# OpenClaw incident analysis

In January 2026, OpenClaw — an open-source agent platform — reached 60,000 GitHub stars in 72 hours and 300,000+ users within weeks. Shortly after, two independent analyses surfaced:

- **Kaspersky** identified **512 vulnerabilities** across the platform and its skills ecosystem, with **8 rated critical**.
- **Cisco** observed **active data exfiltration** in third-party skills installed from the public marketplace.

This page walks through the architectural patterns those findings exposed and assesses, honestly, which LegionForge's design catches — and which it would only catch with operator action — and which it wouldn't catch at all.

!!! note "On scope and tone"
    This isn't a hit piece on OpenClaw. The patterns described below recur across most platforms in the "low-friction agent + skills marketplace" category. The point is to make the architectural tradeoffs concrete, not to score points. LegionForge has its own limits, listed at the bottom.

## Pattern 1: Skills installed without signing

**What it is.** OpenClaw allowed any third party to publish a "skill" (their term for a tool). Users installed skills directly into their agent. There was no signature requirement, no review pipeline, no verification that the skill on disk matched the skill at publish time.

**Why it matters.** A compromised skill author, a hijacked publisher account, or a typosquatted package name → arbitrary code runs in the agent process. Standard supply-chain attack, but with the agent's full capability set.

**LegionForge's response:**

- **Catches it: ✅ Yes.** Every registered tool is Ed25519-signed at registration. The hash check at invocation catches code substitution. The signature check catches unauthorized registration.
- **The honest limit:** if the tool is malicious *at registration time*, LegionForge signs the malicious version. Defense moves up the chain (operator must vet what they register). The differential is: in LegionForge, you sign once and detection runs automatically on every invocation; in the OpenClaw pattern, you trust the publish-time signature, which is exactly what got compromised.

## Pattern 2: Skills with implicit broad capabilities

**What it is.** A skill in the OpenClaw model could declare itself, then call any system API available to the agent process — file system, network, subprocess, environment variables. There was no capability declaration or enforcement.

**Why it matters.** A "weather lookup" skill could read your SSH keys. There's nothing in the architecture saying it can't. The user has no way to know what a skill *will* do — only what it claims to do.

**LegionForge's response:**

- **Catches it: ✅ Yes, by design.** Every tool declares a `required_capability`. Every task has a `capability_scope`. Guardian's check #3 fails the invocation if the tool's capability isn't in the task's scope. A weather lookup tool requiring `FETCH_WEB` cannot do file system reads — even if its code tries.
- **The honest limit:** the tool is still doing whatever its code does *in process*. We don't sandbox tool execution at the OS level. A tool that allocates 100 GB of memory will OOM the process. The capability check stops *outbound effects*, not *internal misbehavior*.

## Pattern 3: Persistent agent privilege

**What it is.** Once an OpenClaw agent was set up with API keys and permissions, those permissions persisted across all tasks. A task that needed `WRITE` for one operation made the agent permanently authorized to write — including for subsequent tasks where write wasn't needed.

**Why it matters.** Capability creep is the default state. An agent compromised mid-life has all accumulated permissions available to the attacker.

**LegionForge's response:**

- **Catches it: ✅ Yes, by design.** Capabilities are *task-scoped*, not *agent-scoped*. A task carries a `capability_scope` array. When the task ends, the scope is gone. The next task starts with whatever scope it declares — typically less.
- **The honest limit:** if the operator habitually submits all tasks with `["WRITE", "EXEC_SHELL_FULL", "DB_WRITE"]` in the scope, they've effectively given the agent persistent privilege. The principle relies on operator discipline.

## Pattern 4: No deterministic security pipeline

**What it is.** OpenClaw relied on the LLM to behave well. There was no pre-execution check that didn't go through a model. If a skill's argument looked suspicious, the only safety net was hoping the LLM would notice.

**Why it matters.** LLM-as-judge for security is the failure mode that defines this category. The same model that's vulnerable to prompt injection is asked to detect prompt injection. The same model that misjudges argument intent decides whether an argument is safe.

**LegionForge's response:**

- **Catches it: ✅ Yes.** Guardian's 7 checks are deterministic. No LLM in the hot path. Decision time is bounded (3-5ms) and predictable. The LLM never makes a security decision in LegionForge.
- **The honest limit:** deterministic checks catch what their patterns express. Novel attack patterns aren't caught until added to `threat_rules`. The advantage isn't omniscience — it's that the patterns are inspectable, auditable, and don't fall to prompt injection themselves.

## Pattern 5: Tool result poisoning

**What it is.** When an OpenClaw skill returned a result, that result was passed back to the LLM with the same trust level as the user's original prompt. A skill that fetched a web page containing prompt-injection payloads → the payload reached the model with full trust.

**Why it matters.** This is the most common live attack against agent systems. Attacker hosts a page with payloads. User asks agent to summarize the page. Payload runs.

**LegionForge's response:**

- **Catches it: ⚠️ Partially, by design.** Tool results re-pass through `sanitize_input()` before re-entering the model context. Tier 1 patterns match → halt with `TOOL_RESULT_INJECTION` event.
- **The honest limit:** Tier 1 catches known patterns. Novel patterns can still reach the model. The *consequence* of a successful injection still has to pass Guardian's checks (the model can be tricked into wanting to do bad things; it still can't do them without capability + Guardian agreement). But the injection *attempt* lands in the model context.

## Pattern 6: No audit trail of consequence

**What it is.** OpenClaw logged HTTP requests and basic actions, but the logs were unstructured, ungranular, and not designed for after-the-fact forensics. Determining what a compromised agent had done was a tedious manual exercise.

**Why it matters.** When a compromise is discovered, the first question is "what did the attacker do?" If the answer is "we don't really know without a lot of work," the breach is functionally unbounded.

**LegionForge's response:**

- **Catches it: ✅ Yes.** Every meaningful event lands in `audit_log` with a SHA-256 hash chain. Every security-relevant event lands in `threat_events` with structured payloads. Forensics is a SQL query, not a log-grep marathon.
- **The honest limit:** if the attacker has DB admin credentials, they can rewrite history (the chain detects the rewrite but doesn't prevent it). Cold storage of audit data is the operator's responsibility.

## Pattern 7: Cloud-augmented model calls without operator visibility

**What it is.** Many OpenClaw skills made cloud API calls (OpenAI, Anthropic, etc.) for embedding lookups or sub-task processing. Operators had limited visibility into which skills called which APIs with which content.

**Why it matters.** A skill that "needs to summarize" can quietly forward arbitrary content to a third-party API. Data exfiltration via API misuse.

**LegionForge's response:**

- **Catches it: ⚠️ Partially.** All LLM calls go through `llm_factory`, which logs to `api_usage`. Every cloud call is recorded. Operators can see what was sent where.
- **The honest limit:** if a tool *needs* to make an outbound API call as part of its declared function, it has the `FETCH_WEB` capability and the call goes through. We log the call but we don't pre-decide whether the API endpoint is safe. Operator-defined rules in `threat_rules` can block specific destinations.

## Summary

| OpenClaw pattern | LegionForge response |
|---|---|
| Unsigned skills | ✅ Ed25519 signing + hash check on every invocation |
| Implicit broad capabilities | ✅ Capability declaration + scope-enforced at boundary |
| Persistent agent privilege | ✅ Task-scoped, not agent-scoped |
| No deterministic security pipeline | ✅ 7-check pipeline on every tool call |
| Tool result poisoning | ⚠️ Tier 1 patterns + capability boundary as second line |
| No audit trail | ✅ Hash-chained `audit_log` + structured `threat_events` |
| Cloud-augmented opacity | ⚠️ Logged but not pre-filtered without operator rules |

Five caught by design, two partially caught with operator action required. The "partial" categories are the load-bearing parts of LegionForge's honest limits: novel injection patterns and operator-defined outbound API rules.

## What this means for evaluators

If you're choosing between LegionForge and a platform in the OpenClaw / Hermes / row-bot category, the relevant questions:

1. **How tolerant am I of supply-chain risk in third-party tools?** If the answer is "not at all," signing matters.
2. **What's the worst thing my agent could be tricked into doing?** If the answer is "execute commands or delete data," capability scoping matters.
3. **If something does go wrong, can I prove what happened?** If the answer needs to be yes, the audit chain matters.
4. **Am I willing to write a few `threat_rules` entries as new patterns emerge?** That's the LegionForge model.

These are the load-bearing differences. The OpenClaw category optimizes for adoption velocity. LegionForge optimizes for shipping consequential agents to users with valuable data.

Different optimizations for different problems. Pick what fits.
