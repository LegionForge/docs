---
title: LegionForge Documentation
hide:
  - navigation
---

# LegionForge Documentation

**A local-first, security-native AI agent platform.**
Security is enforced in the execution path — not layered on afterward.

This is the central documentation hub for the LegionForge ecosystem: the framework, the Guardian security sidecar, and the tools and apps built on top.

[:material-rocket-launch: Get started with the framework](framework/getting-started.md){ .md-button .md-button--primary }
[:material-shield-check: Guardian quickstart](guardian/index.md){ .md-button }
[:fontawesome-brands-github: LegionForge on GitHub](https://github.com/LegionForge){ .md-button }

---

## What's in the ecosystem

### Core

| Project | What it is |
|---|---|
| [**LegionForge**](framework/index.md) | The framework. Local-first AI agent platform built on LangGraph, with a full security stack baked into every layer. |
| [**Guardian**](guardian/index.md) | Deterministic security sidecar for AI agent frameworks. Drop-in protection against prompt injection, tool poisoning, and capability abuse. |

### Tools

| Project | What it is |
|---|---|
| [llm-valet](tools/llm-valet.md) | Cross-platform LLM lifecycle manager — auto-pause/resume Ollama based on resource pressure and gaming detection. |
| [mcp-probe](tools/mcp-probe.md) | Connectivity and configuration advisor for MCP services you own or operate. |
| [headroom](tools/headroom.md) | System stability monitor — memory pressure, paging, AI-powered diagnostics. |
| [hermes-tool-test-suite](tools/hermes-tool-test-suite.md) | pytest harness for validating Hermes AI agent tool-calling reliability. |
| [dev-rig](tools/dev-rig.md) | Shared Python CI workflows, pre-commit config, and audit harness. |

### Apps

| Project | What it is |
|---|---|
| [Jeli](apps/jeli.md) | LegionForge's sovereign, cryptographically-attested personal memory framework. |
| [ConvoBox](apps/convobox.md) | Local, backend-agnostic voice frontend for CLI coding agents. |
| [ADHD-OS](apps/adhd-os.md) | Personal assistant framework for those with ADHD. |

---

## Design principles

The whole LegionForge ecosystem is built around a small set of non-negotiable principles:

- **Fail-safe tiering** — halt → sandbox/retry → degrade. Never silently succeed.
- **Human gates on all mutations** — destructive actions cross a human-in-the-loop boundary.
- **Replace AI with determinism wherever possible** — the LLM is the last resort, not the first.
- **Validate at trust boundaries, not at processing nodes** — sanitize once, at the edge.
- **Privilege tied to tasks, not persistent to agents** — capability-scoped, expires when the task ends.

Read more in [Security Model](framework/security-model.md).

---

## Where to go next

- **New here?** Start with [Framework → Getting Started](framework/getting-started.md).
- **Want to drop Guardian into an existing agent stack?** See [Guardian → Architecture](guardian/architecture.md).
- **Want to contribute?** See [Contribute](contribute/index.md).
- **Looking for the website?** [legionforge.org](https://legionforge.org/).
