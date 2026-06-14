# Differentiators

A technical comparison of LegionForge against the two adjacent categories: **cloud agent platforms** (OpenAI Operator, Anthropic Computer Use, Google Mariner) and **OSS agent frameworks** (LangChain, AutoGen, CrewAI, OpenClaw, Hermes). Not a marketing chart — a security-architecture comparison.

## At a glance

| Dimension | LegionForge | Cloud agent platforms | OSS frameworks |
|---|---|---|---|
| **Where it runs** | Your hardware | Their hardware | Your hardware |
| **Where data sits** | Your PostgreSQL | Their database (opaque) | Wherever you wire it |
| **Tool-call security** | 7-check pipeline enforced on every call | Vendor-internal | What you wire (often nothing) |
| **Prompt injection detection** | 29 patterns, two tiers, at boundary | Vendor-defined | Not bundled |
| **Audit trail** | SHA-256 hash-chained `audit_log` | Their logs (you don't get them) | Not bundled |
| **HITL on destructive actions** | Enforced via approval gate | Sometimes | You wire it |
| **Tool signing** | Ed25519 on every registered tool | Internal | Not bundled |
| **Multi-tenancy isolation** | DB role separation (5 roles) | Vendor-managed | You wire it |
| **License** | AGPL-3.0 + commercial · Guardian MIT | Proprietary | MIT / Apache |

## What changes when you switch categories

### From a cloud agent platform → LegionForge

The big shifts:

- **Data residency.** Your prompts, tool outputs, and conversation history stop leaving your machine.
- **Operational burden.** You now run PostgreSQL, Ollama, and Guardian yourself. Cloud platforms handle this for you.
- **Latency profile.** Local LLMs (Ollama llama3.1:8b on M-series Macs) are ~5-15 tokens/sec; cloud APIs are 50-200+. Local is faster on round-trip overhead, slower on raw decode speed.
- **Cost profile.** Hardware capex + electricity vs. per-token billing. Crosses over at moderate volume.
- **Audit and compliance.** You get the audit log, the hash chain, the threat events. You can prove what happened. Cloud platforms can't (or won't) give you the same visibility.

The platform that's *also* trying to occupy the LegionForge quadrant is rare. Most local agent setups are unguarded — they trade cloud-residency for vendor-trust, but they don't add security in exchange.

### From an OSS framework (LangChain / AutoGen / CrewAI) → LegionForge

The big shifts:

- **Less flexibility.** LegionForge has opinions. The orchestrator graph is a specific shape; tools must be registered with capabilities; mutations require HITL. If you're building a research prototype where flexibility matters more than guardrails, the OSS frameworks are easier.
- **More enforcement.** Guardian's checks aren't optional. The audit log writes whether you want it or not. Loop protection fires whether you configured it or not. The "safe by default" stance is a feature when you ship to production, a friction when you're prototyping.
- **Database dependency.** LegionForge requires PostgreSQL. OSS frameworks can run with in-memory state. PostgreSQL gives us the audit chain, the LangGraph checkpoints, the threat rule hot-reload — but it's a real dependency to operate.
- **Threat model.** LegionForge has one; OSS frameworks largely leave that to you. If you're building a public-facing agent, that's a feature. If you're building an internal tool with low blast radius, it's overhead.

The right framing: **OSS frameworks are substrates; LegionForge is a hardened platform.** The substrate is more flexible but you bring the hardening. The platform is less flexible but the hardening is included.

## On the OpenClaw / Hermes / row-bot generation

A wave of agent platforms launched in late 2025 / early 2026, several reaching tens of thousands of GitHub stars in days. Their general pattern:

- **Low-friction install** — usually a one-line installer or Docker image
- **Skills marketplace** — third-party tools / "skills" that anyone can publish
- **Cloud-augmented** — local execution with cloud LLM calls
- **Light or no security model** — auth is API-key-style, tool calls are trust-the-LLM

This is the right shape for adoption velocity. It's the wrong shape for shipping to users with valuable data.

The Jan 2026 OpenClaw analysis — 512 vulnerabilities flagged by Kaspersky, active data exfiltration in third-party skills found by Cisco — isn't a story about OpenClaw being uniquely bad. It's a story about the **category** being structurally vulnerable. Skills marketplaces with no signing, no capability scoping, and no per-call security checks will always have this profile. The same pattern will repeat across every platform in this category.

See [OpenClaw incident analysis](openclaw-incident.md) for the specific architectural patterns and which would have been caught by LegionForge's design — and which wouldn't have.

## Where LegionForge specifically differs

The five concrete properties that distinguish LegionForge:

1. **Capability-scoped tool calls.** A task's capability scope is set at submission and never widens. Most other frameworks have no scope concept; the agent can call any registered tool.
2. **Cryptographically signed tools.** Every tool is Ed25519-signed at registration. Supply-chain substitution is detected at invocation. No other agent framework we've found does this by default.
3. **Deterministic pre-call security pipeline.** Guardian's 7 checks run in ~3-5ms on every tool call. No LLM in the hot path. Predictable, auditable, fast.
4. **Hash-chained audit log.** Every action is in the log. Tampering breaks the chain. Compliance teams can verify integrity in one pass.
5. **HITL approval gate baked in.** Destructive actions cross a human-in-the-loop boundary by default. You can relax it per-tool; you can't relax it by accident.

Other frameworks have one or two of these in libraries you can add. LegionForge has all five enforced and on by default.

## Where we *don't* differ

To stay honest:

- **The LLMs we use are the same LLMs everyone uses.** llama3.1, qwen2.5, Claude, GPT — we don't have proprietary models with better behavior.
- **The fundamental adversarial robustness of LLMs is the same.** A model that can be prompt-injected by anyone can be prompt-injected by us. The difference is what we do with that knowledge.
- **The MCP, A2A, and protocol layers are standard.** We implement them; we don't replace them.
- **The base graph runtime is LangGraph.** Bugs in LangGraph affect us.

The differentiators are in the **wrapper**, not the components.

## Implications for an evaluator

If you're choosing an agent platform, the questions to ask:

1. **Where does my data live?** Cloud platforms: theirs. OSS frameworks: yours, if you wire it. LegionForge: yours, by default.
2. **Can I prove what an agent did, in a way a compliance auditor would accept?** Cloud platforms: rarely. OSS frameworks: only if you wired logging. LegionForge: yes, via the hash-chained audit log.
3. **What happens when a tool I trusted gets compromised?** Cloud platforms: you don't know until they tell you. OSS frameworks: you don't notice until something breaks. LegionForge: you revoke the tool ID, the cache reloads, and every running agent stops calling it within 10 seconds.
4. **What happens when an agent decides to delete files?** Cloud platforms: depends on vendor. OSS frameworks: it deletes the files. LegionForge: the HITL gate pauses, a human approves or denies, the audit log records the human's decision.

These are the load-bearing differences. Everything else (latency, cost, ergonomics) is operational tuning.
