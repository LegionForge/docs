# Jeli

**Security and governance layer for sovereign personal memory systems.**

[:fontawesome-brands-github: github.com/LegionForge/jeli](https://github.com/LegionForge/jeli){ .md-button .md-button--primary }

## What it is

Jeli is a security and governance layer for personal AI memory systems. It keeps facts, decisions, preferences, history, and agent-written context under your control, with a verifiable chain of custody for every memory.

It does not promise perfect security or tamper-proof memory. Jeli makes poisoning and tampering harder, more visible, and easier to audit through configurable defense layers.

## Status

Current project status: **v0.2.0-alpha**.

The three-branch governance model is implemented and tested: Scoped MCP access, hash-chained writes, constitutional rules, judicial conflict resolution, entity graph extraction, ingestion bouncer, and memory portability.

Jeli has been deployed on local hardware since v0.1.0-alpha.

## Governance model

Jeli models memory governance as separation of powers:

| Branch | Role |
|---|---|
| Executive | Agents propose memories through the scoped MCP server. |
| Legislative | PostgreSQL + pgvector hold append-only, hash-chained memory and state logs. |
| Judicial | Conflict daemons arbitrate contradictions, record precedent, and escalate unresolved cases. |
| Constitutional | User-signed rules block writes, cap trust, filter reads, and define non-negotiable memory policy. |

Agents never get direct database access and cannot claim user-tier authority.

## Current capabilities

- **Hash-chained writes.** Every memory write is HMAC-signed and linked to the previous record; `jeli verify` walks the chain and reports integrity breaks.
- **Scoped MCP server.** Agents can call explicit memory tools without arbitrary shell, filesystem, or database access.
- **Trust-scored provenance.** User-direct, user-confirmed, agent-inferred, external, and flagged content carry different trust ceilings.
- **Layered injection defense.** Regex checks, Unicode normalization, content-class stigma, and an optional LLM classifier help catch prompt-injection-shaped memory writes.
- **Constitutional gates.** User-signed rules can deny writes, cap trust, or filter reads before data leaves the store.
- **Judicial precedent.** Settled contradictions become case law; unresolved conflicts enter a human escalation queue.
- **Ingestion Bouncer.** Proposed memories can stage in an inbox for deduplication, classification, and safe promotion into the hash chain.
- **Entity graph.** Capture auto-extracts people, projects, organizations, technologies, and relations for entity-centric search.
- **Memory portability.** `jeli export` and `jeli import` move memories through JSON-Lines archives with tamper detection and local re-embedding.
- **Local-first embeddings.** Ollama is the default embedding provider; cloud providers are explicit opt-in.

## Memory lifecycle

Jeli treats memory as governed state, not a mutable note field. Updates are append-only state events:

- **Revise** when a memory changes.
- **Invalidate** when a fact is no longer true.
- **Redact** when content should no longer be revealed.
- **Resolve** when memories conflict.
- **Export** when you want to leave or migrate.

The chain preserves auditability while still giving the user practical control over what the system remembers and exposes.

## Integration with LegionForge

Jeli exposes an MCP server that LegionForge agents can use as a controlled memory interface. The framework can read from Jeli, propose new memories, inspect provenance, and query entity relationships through the scoped tool surface.

## Important limits

Jeli verifies attribution and chain integrity. It does not verify truth. A high-trust memory can still be wrong if the user or a trusted channel entered something wrong. The security posture is defense-in-depth: make misuse harder, make tampering visible, and keep the owner in control.
