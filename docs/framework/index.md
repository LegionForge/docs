# LegionForge Framework

**The hardened, self-hosted alternative to cloud agent platforms.**

LegionForge is an open-source framework for running AI agent systems on your own hardware. It runs local LLMs via [Ollama](https://ollama.com) or cloud APIs (OpenAI, Anthropic), with a full security stack baked into every layer of the execution pipeline — not bolted on afterward.

[:fontawesome-brands-github: github.com/LegionForge/LegionForge](https://github.com/LegionForge/LegionForge){ .md-button .md-button--primary }

## Why it exists

In January 2026, OpenClaw hit 60,000 GitHub stars in 72 hours and 300,000+ users in weeks. Kaspersky found 512 vulnerabilities (8 critical). Cisco found active data exfiltration in third-party skills. LegionForge is built in the opposite order: security first, product on top.

## What's in the box

- **Local-first LangGraph runtime** — agents execute on your hardware; cloud APIs are optional
- **Deterministic security pipeline** — prompt-injection detection, tool-revocation, capability-boundary checks, Ed25519 tool signing, adaptive threat rules
- **Three independent loop-protection layers** — step counter, action-history loop detection, token-budget guard
- **Multi-provider LLM factory** — Ollama, OpenAI, Anthropic, InceptionLabs (mercury-2) behind one interface
- **PostgreSQL state layer** — async pool, LangGraph checkpoint resumption, pgvector RAG, threat-event audit chain
- **Gateway API** — FastAPI with task queue, SSE streaming, A2A + MCP endpoints, Bearer auth
- **Connectors** — Discord, Telegram, Slack, WhatsApp, webhooks
- **Web UI** — task submission, session continuity, multi-modal image input, HITL approval gate
- **Guardian sidecar** — 7-check deterministic pipeline running as a separate process

## Pages in this section

<div class="grid cards" markdown>

-   :material-rocket-launch: **[Getting Started](getting-started.md)**

    Install, set up the venv, configure the hardware profile, run a smoke test, send your first task through the gateway.

-   :material-puzzle: **[Architecture](architecture.md)**

    Core design principles. Module map. How a task flows from gateway → orchestrator → agents → tools → response.

-   :material-shield-lock: **[Security Model](security-model.md)**

    Where validation happens, what threats it catches, how Guardian fits in.

-   :material-cog: **[Configuration](configuration.md)**

    The hardware profile system. Pydantic settings singleton. Switching between profiles.

-   :material-database: **[Database](database.md)**

    PostgreSQL schema, role separation, LangGraph checkpoints, pgvector RAG, audit chain.

-   :material-api: **[Gateway & API](gateway.md)**

    Task submission, SSE streaming, A2A + MCP endpoints, auth model.

-   :material-link-variant: **[Connectors](connectors.md)**

    Discord, Telegram, Slack, WhatsApp, webhooks. How a bot bridges to the gateway.

-   :material-alert-octagon: **[Threat Events](threat-events.md)**

    Structured event types logged to `threat_events`. What each one means and what fires it.

-   :material-book-open-page-variant: **[API Reference](api-reference.md)**

    Auto-generated reference from the framework's Python source. Coming soon.

</div>
