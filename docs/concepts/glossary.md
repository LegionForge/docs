# Glossary

Every LegionForge-specific term in one place. One-line definitions; the linked pages have the long version.

## A

**A2A (Agent-to-Agent)**
: A LegionForge endpoint pattern that lets one agent invoke another agent's capabilities synchronously, with the same auth + Guardian path as a user request.

**Action history hash**
: An MD5 hash of the last *N* tool-call signatures (default *N* = 5). Used by the second loop-protection layer: if the same signature repeats 3 times in the window, the agent is halted with `LOOP_DETECTED`.

**Adaptive threat rule**
: A row in the `threat_rules` table that adds a new match → action pair to Guardian without requiring a code or process restart. Hot-reloaded every 10 seconds.

**Audit chain**
: The SHA-256 hash chain over rows in the `audit_log` table. Each row's `prev_hash` is the SHA-256 of the previous row's content. Tampering with any row breaks the chain.

## B

**Bearer auth**
: Token-based authentication used by the gateway. Every request carries `Authorization: Bearer <key>`. Keys live in `gateway_users` (hashed, never stored plaintext).

**Boundary** *(trust boundary)*
: A place in the system where data crosses from one trust level to another. LegionForge has exactly three: input, tool execution, output. Validation only happens at boundaries. See [Security fundamentals](security-fundamentals.md).

## C

**Capability**
: A small string naming a class of action (e.g., `READ`, `WRITE`, `FETCH_WEB`, `EXEC_SHELL_SAFE`). Tools declare which capability they require. Tasks declare which capabilities they're authorized to use.

**Capability scope**
: The array of capabilities a task is authorized to use. Set at submission, never widens. See [Tools and capabilities](tools-and-capabilities.md).

**Checkpoint**
: A snapshot of agent state written to `langgraph_checkpoints` after every node. Used to resume a paused or crashed task without redoing prior steps.

**Connector**
: A bridge between an external chat platform (Discord, Telegram, Slack, WhatsApp, generic webhook) and the LegionForge gateway. Translates platform messages into task submissions and streams responses back.

## D

**Destructive pattern detection**
: One of Guardian's 7 checks. A regex pipeline applied to tool call arguments looking for known-dangerous patterns (`rm -rf /`, `DROP TABLE`, fork bombs, etc.).

**Deterministic check**
: A check whose decision is fully determined by its inputs and rules — no LLM involved. The opposite of an LLM-based check. LegionForge prefers deterministic over LLM-based wherever possible.

## E

**Ed25519 signing**
: The public-key signature scheme used to sign tool registrations. The private key (`legionforge_tool_signer`) is in macOS Keychain; the public key is in Guardian's config.

## G

**Gateway**
: The FastAPI service on port 8080 that accepts task submissions, dispatches workers, streams responses via SSE, and serves the web UI. The outside world's entry point.

**Guardian**
: The deterministic security sidecar on port 9766. Runs 7 checks on every tool invocation. See [Guardian](../guardian/index.md).

## H

**HITL (Human-in-the-Loop)**
: An approval gate that pauses a task before a destructive action so a human can review and approve or deny. Mutating tools require HITL by default.

## I

**Injection detection**
: Prompt-injection pattern matching done at `sanitize_input()`. 29 patterns in two tiers — Tier 1 rejects immediately, Tier 2 downgrades trust.

## L

**LangGraph**
: The state-machine framework from LangChain that LegionForge agents are built on. Each agent is a graph; each step is a node.

**`legionforge_admin`**
: The PostgreSQL superuser. Renamed from `jp` in March 2026. Used only for migrations, never at runtime.

**Loop protection**
: Three independent layers that prevent agents from spinning forever — step counter, action-history hash, token budget. All three must pass on every step.

## M

**MCP (Model Context Protocol)**
: An open protocol for connecting LLMs to context sources (databases, APIs, file systems, etc.). LegionForge can act as both an MCP client (consuming external MCP servers) and an MCP server (exposing tools to external MCP clients).

## O

**Orchestrator**
: The main agent in LegionForge — the LangGraph state machine that takes a prompt, plans steps, calls tools, and produces a response. Lives in `src/base_graph.py` as a template.

## P

**PII redaction**
: Pattern-based stripping of personal identifiers (email, SSN, phone, credit card, etc.) from output before logging or returning to the user. Runs at `sanitize_output()`.

**Profile** *(hardware profile)*
: A YAML file in `config/hardware_profiles/` that defines memory limits, model selection, safeguard thresholds, and paths for a specific machine class. Selected via `AGENT_HARDWARE_PROFILE` env var.

## R

**Rate limiter**
: Pre-execution cost estimator that blocks LLM calls before they exceed the per-task or per-provider budget. See `src/rate_limiter.py`.

**Revoked tool**
: A tool ID added to the `revoked_tools` table after a vulnerability or deprecation is discovered. Guardian's cache picks it up within 10 seconds; subsequent calls are denied without redeploying any code.

## S

**Safeguards**
: The collective name for LegionForge's three loop-protection layers and the rate limiter — anything that prevents an agent from over-running its budget.

**Sanitize input / sanitize output**
: The two functions in `src/security/core.py` that run at trust boundaries 1 and 3 respectively. `sanitize_input()` runs injection detection; `sanitize_output()` runs PII redaction.

**SecureToolNode**
: The framework wrapper around tool dispatch that calls Guardian before invoking the actual tool function. Catches Guardian denials and surfaces them as `ToolMessage` results to the LLM.

**Sequence contract**
: A configured ordering rule that some tool calls must follow (e.g., `delete_file` must be preceded by `read_file`). Enforced by Guardian as one of the 7 checks.

**Sidecar**
: A separate process running alongside the main application, providing a specific capability over IPC or HTTP. Guardian is a sidecar to the framework.

**SSE (Server-Sent Events)**
: The streaming protocol the gateway uses to deliver task progress (tool calls, intermediate results, final response) back to the client in real time.

## T

**Task**
: One unit of work submitted to the gateway. See [Agents and tasks](agents-and-tasks.md).

**Threat event**
: A row in the `threat_events` table recording a security-relevant occurrence (injection detected, tool hash mismatch, loop detected, etc.). See [Threat Events](../framework/threat-events.md).

**Token budget**
: The maximum total tokens a task may use. Cumulative across all LLM calls in the task. Alert at 80%, force-end at 100%. The third layer of loop protection.

**Tool**
: A function the LLM can ask to invoke. Registered with the framework, hashed, signed, capability-scoped. See [Tools and capabilities](tools-and-capabilities.md).

**Tool signing**
: Cryptographic signing (Ed25519) of registered tool code hashes. Catches supply-chain attacks where a dependency upgrade replaces a tool's implementation.

**Trace ID**
: A UUID generated by the gateway middleware and threaded through to LangSmith, Guardian, and the worker. Used to correlate logs across components.

## V

**Verify node**
: The final node in the orchestrator graph. Re-checks the answer for obvious issues (truncation, off-topic, etc.) and may re-prompt the LLM once (`MAX_VERIFY_ROUNDS=1`).
