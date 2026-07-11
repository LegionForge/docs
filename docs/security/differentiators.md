# Differentiators

A technical comparison of LegionForge against two adjacent categories: **cloud agent platforms** and **OSS agent frameworks**. The point is not that those projects are bad. Many of them are excellent and pushed the whole field forward. The question is simpler: who owns the memory, who can audit the system, and who decides what an agent is allowed to do?

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
| **License** | Open source, project-specific licenses | Proprietary | MIT / Apache |

## What changes when you switch categories

### From a cloud agent platform -> LegionForge

The big shifts:

- **Custody.** Prompts, tool outputs, memory, and audit trails can stay on infrastructure you control.
- **Operational burden.** You now run PostgreSQL, Ollama, and Guardian yourself. Cloud platforms handle this for you.
- **Latency profile.** Local LLMs (Ollama llama3.1:8b on M-series Macs) are ~5-15 tokens/sec; cloud APIs are 50-200+. Local is faster on round-trip overhead, slower on raw decode speed.
- **Cost profile.** Hardware capex + electricity vs. per-token billing. Crosses over at moderate volume.
- **Audit and compliance.** You get the audit log, the hash chain, the threat events, and the memory provenance trail. You can inspect what happened without waiting for a vendor export.

The tradeoff is real: you accept more operational responsibility in exchange for more custody and auditability.

### From an OSS framework (LangChain / AutoGen / CrewAI) -> LegionForge

The big shifts:

- **Less flexibility.** LegionForge has opinions. Tools carry capabilities, mutations can require human gates, and memory carries provenance. If you are building a research prototype where flexibility matters more than guardrails, general-purpose frameworks are easier.
- **More enforcement.** Guardian checks, audit logs, loop protection, trust scoring, and memory governance are not left as afterthoughts. That stance is useful when you care about long-lived agents and personal memory; it is friction when you only need a quick prototype.
- **Database dependency.** LegionForge requires PostgreSQL. OSS frameworks can run with in-memory state. PostgreSQL gives us the audit chain, the LangGraph checkpoints, the threat rule hot-reload — but it's a real dependency to operate.
- **Threat model.** LegionForge has one; OSS frameworks largely leave that to you. If you're building a public-facing agent, that's a feature. If you're building an internal tool with low blast radius, it's overhead.

The right framing: **OSS frameworks are substrates; LegionForge is a hardened platform.** The substrate is more flexible but you bring the hardening. The platform is less flexible but the hardening is included.

## On the fast-moving agent generation

A wave of agent platforms launched in late 2025 / early 2026, several reaching large audiences quickly. That work matters: it proved there is real demand for agentic software that is easier to install, extend, and use.

It also surfaced a structural lesson. Low-friction agents, third-party skills, local execution, cloud model calls, and broad tool access create a lot of power very quickly. Without signing, scoped capabilities, provenance, and per-call checks, the same openness that makes a platform useful can also make it hard to trust.

LegionForge's response is not to reject that generation of tools. It is to add the parts that make long-lived, user-owned systems safer to operate: custody, auditability, deterministic guardrails, and human authority over high-impact changes.

See [OpenClaw incident analysis](openclaw-incident.md) for one concrete case study and which architectural patterns LegionForge addresses, partially addresses, or does not claim to solve.

## Where LegionForge specifically differs

The concrete properties that distinguish LegionForge:

1. **User-owned memory and audit state.** Memory, provenance, and audit trails are designed to live under the user's custody.
2. **Capability-scoped tool calls.** A task's capability scope is set at submission and should not silently widen.
3. **Cryptographically signed tools.** Registered tools can be checked for supply-chain substitution at invocation.
4. **Deterministic pre-call security pipeline.** Guardian checks run before tool calls. No LLM in the hot path.
5. **Hash-chained audit and memory provenance.** Actions and memories can be verified for tampering.
6. **Human authority baked in.** Destructive actions, sensitive memory writes, and unresolved conflicts can cross a human-controlled boundary.

Other frameworks may support some of these with libraries or custom wiring. LegionForge makes them part of the platform posture.

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
