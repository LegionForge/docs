# Briarios

**Security-native orchestration meta-layer for parallel agent workstreams.**

[:fontawesome-brands-github: github.com/LegionForge/Briarios](https://github.com/LegionForge/Briarios){ .md-button .md-button--primary }

## What it is

Briarios sits above agent execution systems and adds controls for running parallel development lanes safely: security-aware triage, model-tier routing, multi-resource budget gates, and independent verification.

It does not replace OpenHands, LangGraph, or managed agent platforms. It uses those systems as execution layers and supplies the policy and verification layer around them.

## What it adds

- **Security-aware triage.** Issues are scored by security risk, blast radius, complexity, and blocking impact before assignment.
- **Model-tier routing.** High-risk or ambiguous work routes to stronger models; bounded tasks can use cheaper lanes.
- **Budget gates.** Parallel lanes are gated by token budget, CPU load, VRAM headroom, and external API limits.
- **Independent verification.** Diffs pass through test, review, eval, and security checks before a human merge gate.

## Triage axes

| Axis | What it captures |
|---|---|
| Security risk | Cosmetic change vs. exploitable vulnerability |
| Blast radius | Isolated file vs. cross-cutting subsystem |
| Complexity | Clear implementation vs. architecture decision |
| Blocking | Standalone task vs. blocker for other work |

## Status

Research complete. Architectural decisions are accepted and the project is ready for scaffolding.
