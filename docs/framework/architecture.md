# Architecture

## Core design principles

LegionForge is built around five non-negotiable principles. Every module and every code path is shaped by them:

| Principle | What it means in practice |
|---|---|
| **Fail-safe tiering** | Halt → sandbox/retry → degrade. Never silently succeed. Errors propagate with intent. |
| **Human gates on all mutations** | Destructive actions cross a human-in-the-loop boundary by default. |
| **Replace AI with determinism wherever possible** | The LLM is the last resort, not the first. Rules, tables, and pattern matchers run ahead of model calls. |
| **Validate at trust boundaries, not at processing nodes** | Sanitize once, at the edge. Internal code trusts internal data. |
| **Privilege tied to tasks, not persistent to agents** | Capability is scoped to the active task and expires when the task ends. |

## Key modules

| Module | Responsibility |
|---|---|
| `config/settings.py` | Pydantic singleton loaded from a hardware YAML profile. All memory limits, model names, safeguard thresholds, and paths come from here. |
| `src/base_graph.py` | LangGraph template. Copy this when creating new agents. Wires in three-layer loop protection, token budgeting, per-run tracing toggle, TOCTOU snapshot, and Guardian pre-invocation check automatically. |
| `src/security/core.py` | API key management via macOS Keychain (no `.env` secrets), prompt-injection detection (29 patterns, Tier 1/2 tiering), PII redaction. All inputs pass through `sanitize_input()`; all outputs through `sanitize_output()`. |
| `src/security/guardian.py` | Guardian FastAPI sidecar on port 9766. See [Guardian](../guardian/index.md). |
| `src/safeguards.py` | Three independent loop-protection layers. |
| `src/database.py` | Async PostgreSQL pool (admin + restricted app roles), LangGraph `AsyncPostgresSaver` for checkpoint resumption, pgvector RAG, 16 tables. |
| `src/llm_factory.py` | Unified factory for Ollama, OpenAI, Anthropic, InceptionLabs. Reads config from the hardware profile. Supports cloud fallback. |
| `src/rate_limiter.py` | Per-provider rate limits with pre-execution token cost estimation. Hard daily caps with 80% / 100% alert thresholds. |
| `src/gateway/app.py` | FastAPI gateway on port 8080. Task submission queue, SSE streaming, web UI, A2A + MCP endpoints, Bearer auth. |
| `src/connectors/discord.py` | Discord bot connector. Bridges `!<task>` messages → gateway → SSE stream → reply edits. (And similar for Telegram, Slack, WhatsApp.) |

## Three independent loop-protection layers

A single failure shouldn't let an agent spin forever. Three independent layers must all pass on every step. If any one fires, execution halts and a threat event is logged.

```mermaid
flowchart TB
    Start([Step begins]) --> L1{Step counter<br/>limit reached?}
    L1 -->|yes| H1[HALT<br/>STEP_LIMIT_REACHED]
    L1 -->|no| L2{Action history<br/>signature repeated<br/>3× in last 5 steps?}
    L2 -->|yes| H2[HALT<br/>LOOP_DETECTED]
    L2 -->|no| L3{Token budget<br/>used >= 100%?}
    L3 -->|yes| H3[HALT<br/>TOKEN_BUDGET_EXCEEDED]
    L3 -->|no| Continue([Continue to next step])

    classDef halt fill:#ff4444,stroke:#cc0000,color:#fff
    class H1,H2,H3 halt
```

| Layer | Mechanism | Threshold |
|---|---|---|
| **Step counter** | LangGraph recursion limit | Hard stop on N steps |
| **Action-history** | MD5 hash of the last 5 tool-call signatures | Same signature 3× → halt |
| **Token budget** | Cumulative per-task token usage | Alert at 80%, force-end at 100% |

See [Threat Events](threat-events.md) for the corresponding event types.

## Module map

```mermaid
flowchart TB
    subgraph Edge["Edge layer"]
        Gateway["gateway/app.py<br/>FastAPI :8080"]
        Connectors["connectors/<br/>Discord · Slack · Telegram<br/>WhatsApp · Webhook"]
    end

    subgraph Core["Core layer"]
        Orchestrator["base_graph.py<br/>LangGraph template"]
        Safeguards["safeguards.py<br/>3-layer loop protection"]
        Sanitize["security/core.py<br/>sanitize_input/output"]
        Factory["llm_factory.py<br/>Ollama · OpenAI · Anthropic"]
        Rate["rate_limiter.py<br/>per-provider caps"]
    end

    subgraph GuardianBox["Guardian sidecar"]
        GuardianAPI["security/guardian.py<br/>FastAPI :9766"]
    end

    subgraph Infra["Infrastructure"]
        PG[("PostgreSQL 17<br/>16 tables · pgvector")]
        Ollama["Ollama<br/>llama3.1:8b · qwen2.5:3b"]
        Cloud["Cloud LLMs<br/>OpenAI · Anthropic · InceptionLabs"]
    end

    Connectors --> Gateway
    Gateway --> Orchestrator
    Orchestrator --> Sanitize
    Orchestrator --> Safeguards
    Orchestrator --> Factory
    Factory --> Rate
    Factory --> Ollama
    Factory --> Cloud
    Orchestrator -.->|every tool call| GuardianAPI
    GuardianAPI <--> PG
    Gateway <--> PG
    Orchestrator <--> PG
```

## Request flow

A task submitted to the gateway flows through gateway → worker → orchestrator → Guardian → LLM → tools → response, with checkpoints written along the way so a paused task can be resumed.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant G as Gateway<br/>(:8080)
    participant W as Worker
    participant O as Orchestrator
    participant Gr as Guardian<br/>(:9766)
    participant L as LLM
    participant T as Tool
    participant DB as PostgreSQL

    User->>G: POST /tasks (Bearer)
    G->>G: Authenticate
    G->>W: Enqueue
    W->>O: run_orchestrator()
    O->>O: sanitize_input()
    O->>DB: checkpoint

    loop For each step (bounded by safeguards)
        O->>L: LLM call (rate-limited)
        L-->>O: tool_calls
        O->>Gr: POST /check
        Gr->>DB: log threat_events (async)
        Gr-->>O: allow / deny
        O->>T: invoke (if allowed)
        T-->>O: result
        O->>DB: checkpoint
    end

    O->>O: sanitize_output()
    O-->>W: result
    W-->>G: SSE events
    G-->>User: stream
    G->>DB: audit_log
```

The orchestrator never trusts the LLM. Every tool call passes through Guardian; every input and output crosses a sanitization boundary; every step is checkpointed so a failure mid-task is recoverable.

## Infrastructure dependencies

| Component | Purpose |
|---|---|
| **PostgreSQL 17** | Database: `legionforge`. Password in macOS Keychain (`service: postgres`). |
| **Ollama** | Local LLM runtime. Primary: `llama3.1:8b`. Router: `qwen2.5:3b`. Embeddings: `mxbai-embed-large`. |
| **Docker Desktop** | Required for Guardian sidecar. |
| **macOS Keychain** | All secrets. Never `.env` for production keys. |

## Phase status

- **Phases 0–16** — Full security stack, multi-user gateway, integration tests, modular auth, containerized gateway, multi-provider auth registry, Redis-backed state layer, Kerberos GSSAPI backend, multi-instance docker-compose, Redis global budget counters, Prometheus /metrics endpoint, request trace ID middleware, polished web UI, Telegram/Slack/Webhook channel connectors.
- **Phases 60–381 + G1–G4 + H + I + J + HITL** — 381-tool operator dashboard, web_fetch_js headless browser, Guardian G1–G4 (PyPI published, public repo live, auto-sync Action), agent memory, dual license (AGPL-3.0 + commercial), session continuity UI, multi-modal image input, HITL approval gate, WhatsApp connector.

Current test baseline:

| Suite | Count |
|---|---|
| Smoke | 2247 |
| Integration | 38 |
| Kerberos live-KDC | 5 |
| UI (Playwright) | 40 |
| TestLab | 104 |
| Tool accuracy | 79 |
| Crystallization | 114 |
