# idn-analyzer

**Offline prompt profiler for LLM routing decisions.**

[:fontawesome-brands-github: github.com/LegionForge/Intelligence-Delivery-Network-Request-Analyzer](https://github.com/LegionForge/Intelligence-Delivery-Network-Request-Analyzer){ .md-button .md-button--primary }

## What it is

idn-analyzer analyzes a natural-language prompt before any LLM call is made and returns structured metadata about complexity, domain, PII risk, routing tier, and execution requirements.

It is the analytical core of the [Intelligence Delivery Network](../research/intelligence-delivery-network.md), but it is designed to be independently useful in any LLM application, gateway, or router.

## Example output

```json
{
  "token_estimate": 312,
  "complexity_score": 0.74,
  "reasoning_hops": 2,
  "domain_tags": ["coding"],
  "pii_risk": "none",
  "data_egress_permitted": true,
  "recommended_tier": "L2",
  "confidence": 0.91
}
```

## Design goals

- **Offline-capable.** Layer 1 analysis is rules-based and can run without a network call or model invocation.
- **PII-aware.** Prompts can be flagged for sensitive data and compliance constraints before cloud egress.
- **Gateway-agnostic.** The output schema is meant to feed custom routers, LiteLLM-style proxies, or app-level policy engines.
- **Cross-platform.** The roadmap includes Python, JavaScript, C/C++, WebAssembly, mobile, and embedded targets.

## Status

Development is currently on hold. The repository captures the pre-alpha design, schema, CLI shape, and platform roadmap.
