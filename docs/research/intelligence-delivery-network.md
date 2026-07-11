# Intelligence Delivery Network

**CDN-style routing model for LLM workloads.**

[:fontawesome-brands-github: github.com/LegionForge/Intelligence-Delivery-Network](https://github.com/LegionForge/Intelligence-Delivery-Network){ .md-button .md-button--primary }

## What it is

Intelligence Delivery Network, or IDN, is a CDN-inspired routing architecture for AI workloads. Instead of sending every request to the same large model, IDN profiles the request first and routes it to the compute tier that fits its complexity, privacy needs, latency target, and tool requirements.

The goal is to keep simple, private, and latency-sensitive work close to the user while reserving global frontier models for requests that actually need them.

## Tier model

| Tier | Location | Typical use |
|---|---|---|
| L0 | On-device | Offline use, privacy-forced prompts, PII preprocessing |
| L1 | Edge | Fast extraction, classification, lightweight RAG |
| L2 | Regional | Coding, summarization, document workflows, tool agents |
| L3 | Global | Deep reasoning, research synthesis, multi-agent planning |

## Core components

- **Workload profiler.** Analyzes token size, complexity, reasoning hops, domain, PII risk, and latency sensitivity.
- **Router.** Maps profiler output to a tier, expert model, tool plan, fallback path, and cost estimate.
- **L0 privacy firewall.** Keeps data on-device when egress is blocked and can redact sensitive fields before escalation.
- **Async planner.** Splits multi-part requests into subtasks that can route to different tiers and merge cleanly.

## Relationship to idn-analyzer

[idn-analyzer](../tools/idn-analyzer.md) is the standalone prompt profiler that supplies the routing metadata IDN consumes.

## Status

Development is currently on hold. The repository captures the architecture, routing model, profiler schema, and prototype direction.
