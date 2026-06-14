# Security fundamentals

LegionForge's security model rests on one thesis: **the LLM is not trustworthy**. This page unpacks what that means, why it matters, and how it shapes every component you'll work with.

## The thesis

Every output from an LLM is, in security terms, **attacker-controlled data**. Even if the user is benign, the *prompt context* the LLM operates on includes:

- The user's prompt (which they may craft adversarially)
- Tool results (which may contain injection payloads from external sources)
- Conversation history (which may be partially adversarial)
- Memory (which may have been poisoned in earlier turns)

The LLM cannot reliably distinguish "this fetched web page is content I should summarize" from "this fetched web page contains instructions telling me to call `delete_file` on the user's home directory." Models do this badly. Even the frontier models do this badly. There is no current technique that makes them do it reliably.

The right response is to **stop asking the LLM to be the security boundary**. The boundary is enforced in deterministic code that runs before and after the LLM ever sees the data.

## Trust boundaries

A trust boundary is a place where data crosses from one trust level to another. LegionForge has exactly three:

```mermaid
flowchart LR
    UserIn([User input]) --> B1{{Boundary 1<br/>sanitize_input()}}
    B1 --> Internal["Internal processing<br/>(trusts data)"]
    Internal --> B2{{Boundary 2<br/>Guardian}}
    B2 --> Tools["Tool execution"]
    Tools --> Internal
    Internal --> B3{{Boundary 3<br/>sanitize_output()}}
    B3 --> UserOut([User output])

    classDef boundary fill:#0d1117,stroke:#00ff88,color:#00ff88,stroke-width:2px
    class B1,B2,B3 boundary
```

**Boundary 1: Input.** `sanitize_input()` runs the prompt through 29 prompt-injection patterns. Tier-1 matches reject immediately. Tier-2 matches downgrade trust.

**Boundary 2: Tool execution.** Guardian's 7 checks run on every tool call. Anything denied does not execute.

**Boundary 3: Output.** `sanitize_output()` strips PII before logging or returning to the user.

Validating at boundaries (not at processing nodes) is the only sustainable model. If you try to validate everywhere, every node has to know about every threat, and a missed validation slips through unnoticed. With boundaries, there are exactly three places to audit.

## Replace AI with determinism

The second principle: when there's a choice between an LLM-based check and a deterministic check, pick deterministic every time.

Examples in LegionForge:

| Need | LLM-based approach | LegionForge's approach |
|---|---|---|
| Is this prompt malicious? | Ask the model | Regex pipeline against 29 known patterns |
| Is this tool call safe? | Ask the model | Guardian's 7 deterministic checks |
| Is this output PII? | Ask the model | Pattern matchers for emails, SSNs, phone numbers, etc. |
| Is this argument destructive? | Ask the model | Regex against `rm -rf /`, `DROP TABLE`, etc. |
| Is this loop diverging? | Ask the model | MD5 hash of action signature, count in window |

Deterministic is **slower to extend** (you have to add new patterns) but it's **faster to run**, **predictable in cost**, and **adversarially robust** (an attacker can't talk it out of its decision).

The LLM is the last resort, not the first.

## Why a sidecar (Guardian)

The choice to put Guardian in a separate container, not a Python library, comes from three properties we wanted:

1. **Crash isolation.** A bug in the security layer shouldn't be able to crash the agent. (And vice versa.) The blast radius of either failure stays bounded.

2. **Framework independence.** Guardian works with LegionForge but it's not LegionForge-specific. The HTTP interface means it drops into LangChain, AutoGen, CrewAI, or a custom agent loop. The choice is the same in each: do you call Guardian before invoking a tool, or do you not? Most frameworks don't.

3. **Hot-reloadable rules.** Guardian polls the `threat_rules` table every 10 seconds. When you discover a new attack pattern (or a CVE drops in a tool you've registered), you add a rule to PostgreSQL. Within 10 seconds, the rule is live across every agent process in your deployment. No restart. No redeploy. No coordination.

## Failure modes the model gets right

To be clear about what *is* and *isn't* protected:

LegionForge's deterministic checks catch:

- **Pattern-matchable injection** — the patterns we have, and the patterns you add as you discover them
- **Tool supply-chain attacks** — via hash + signature verification at invocation
- **Capability creep** — via task-scoped capability boundaries
- **Runaway agents** — via three independent loop-protection layers
- **Destructive argument injection** — via regex over tool args
- **Sequence violations** — via configured `sequence_contracts`
- **Tool result injection** — via re-running input sanitization on tool outputs before they re-enter the model's context

What we cannot prevent:

- **Novel injection patterns we haven't seen** — though the adaptive `threat_rules` table makes adding them cheap
- **A compromised LLM weight file** — model integrity is checked at load, not at every inference
- **A malicious operator with gateway credentials** — Bearer auth gates entry; we assume the operator is authorized
- **Physical access to the machine** — out of scope

We list both because a security model that claims to defend against everything is one nobody has actually walked.

## What's next

- **[Glossary](glossary.md)** — quick reference for the vocabulary
- **[Framework → Security Model](../framework/security-model.md)** — the technical version of this page
- **[Guardian → Architecture](../guardian/architecture.md)** — how the sidecar actually works
- **[Security → Threat Model](../security/threat-model.md)** — the kill-chain view of attacks → defenses
