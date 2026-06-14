# Jeli

**LegionForge's sovereign, cryptographically-attested personal memory framework.**

[:fontawesome-brands-github: github.com/LegionForge/jeli](https://github.com/LegionForge/jeli){ .md-button .md-button--primary }

## What it is

Jeli is a personal memory framework — a sovereign store of facts, decisions, preferences, and history that an AI assistant can read from and contribute to. "Sovereign" means it lives on your hardware, under your keys; not in a SaaS database somewhere. "Cryptographically attested" means every entry is signed at write time, so tampering and forgery are detectable.

The model: your AI assistant queries Jeli before answering, augmenting the model's context with facts only you have. New facts learned during a conversation get proposed back to Jeli for review and signing — humans-in-the-loop on memory persistence.

## Status

Active. Public.

## Design principles

The same principles that shape LegionForge shape Jeli:

- **Local-first.** Memory lives on your machine. Replication is opt-in and explicit.
- **Cryptographic provenance.** Every entry has a signature. Memory can be audited and replayed.
- **Human gates on writes.** New memories are *proposed* by the assistant and *committed* by you. Same pattern as HITL on destructive tool calls.
- **Replace AI with determinism wherever possible.** Memory retrieval is structured (tags, types, dates) before falling back to embedding search.

## Use cases

- Personal AI assistants that need stable, user-controlled context
- Long-running agents that should accumulate knowledge across sessions
- Compliance scenarios where memory provenance has to be auditable

## Integration with LegionForge

Jeli exposes an MCP server. The LegionForge framework can register Jeli as a context source and gain read/propose access to memory through the standard MCP path. The HITL approval gate in LegionForge handles the "approve this proposed memory" step.
