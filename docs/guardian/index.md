# Guardian

**A deterministic security sidecar for LLM agent frameworks.**

Guardian is a FastAPI service that runs in a Docker container alongside your agent stack. Every tool invocation is gated through Guardian's 7-check pipeline before it can execute. The checks are deterministic — no LLM in the hot path — so latency is predictable (sub-5ms in practice).

[:fontawesome-brands-github: github.com/LegionForge/guardian](https://github.com/LegionForge/guardian){ .md-button .md-button--primary }
[:fontawesome-brands-python: PyPI: legionforge-guardian](https://pypi.org/project/legionforge-guardian/){ .md-button }

## Why a separate sidecar

Three reasons Guardian is a separate process rather than a Python library imported into the agent framework:

1. **Crash isolation.** A bug in Guardian shouldn't take down the agent, and vice versa.
2. **Framework independence.** Guardian works with LegionForge but it's not LegionForge-specific. It's designed to drop into any agent framework — LangChain, LangGraph, AutoGen, CrewAI, custom — via HTTP.
3. **Hot-reloadable rules.** Guardian polls the `threat_rules` table every 10 seconds. New rules go live without restarting any agent process.

## The 7 checks

In order, every tool invocation must pass:

1. **Tool revocation list** — is the tool ID on the revocation list?
2. **Hash validation** — does the loaded tool's code hash match the registered hash?
3. **Capability boundary** — does this task's capability scope include this tool?
4. **Destructive pattern detection** — do the arguments contain known destructive patterns (`rm -rf /`, `DROP TABLE`, etc.)?
5. **Sequence contracts** — has the agent violated an ordering rule (e.g., "must call `read` before `delete`")?
6. **Ed25519 signature verification** — is the tool's stored signature valid for its stored hash?
7. **Adaptive threat rules** — does the call match any rules loaded from `threat_rules`?

Any check that fails → denied. The denial is logged to `threat_events` as `GUARDIAN_DENIED` with `payload.check_name` identifying which check failed.

[Read the full check list →](checks.md)

## Pages in this section

<div class="grid cards" markdown>

-   :material-puzzle: **[Architecture](architecture.md)**

    How Guardian fits next to your agent framework, the request flow, the rule storage, the hot-reload mechanism.

-   :material-check-circle: **[Checks](checks.md)**

    Detail on each of the 7 checks: what catches it, what payload data it expects, how to extend it.

-   :material-docker: **[Deployment](deployment.md)**

    Docker-compose configuration, environment variables, database role setup, health/metrics endpoints.

</div>

## Status

| | |
|---|---|
| **Version** | 0.1.0 (PyPI) |
| **License** | MIT (independent of the framework's AGPL-3.0) |
| **Auto-sync** | Public repo auto-syncs from the framework's private dev branch |
