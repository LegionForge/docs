# Security

The Security section is for people evaluating LegionForge against other agent platforms with a security lens. The Framework and Guardian sections cover *how* LegionForge enforces security. This section covers *why* it differs from the others and *what* an attacker actually faces.

## Pages

<div class="grid cards" markdown>

-   :material-shield-alert: **[Threat Model](threat-model.md)**

    STRIDE applied to agent systems. The kill-chain view: attacker goal → vector → which LegionForge layer catches it (or honestly, doesn't).

-   :material-compare: **[Differentiators](differentiators.md)**

    Side-by-side: LegionForge vs cloud agent platforms (Operator, Computer Use, Mariner) vs unguarded OSS frameworks (LangChain, AutoGen, CrewAI). Technical, not marketing.

-   :material-magnify-scan: **[OpenClaw incident analysis](openclaw-incident.md)**

    What the Jan 2026 OpenClaw incident actually exposed. Which patterns LegionForge's architecture catches; which it would have caught only with operator action; which it wouldn't catch at all. Honest.

</div>

## Where to go from here

- For the *technical* security model — what runs where, what the deterministic checks look like — see [Framework → Security Model](../framework/security-model.md).
- For Guardian's specific behavior — the 7 checks, the rule storage, the canary endpoint — see [Guardian → Architecture](../guardian/architecture.md).
- For an introduction to the *thinking* behind the model — the LLM-is-not-trustworthy thesis, why deterministic, why sidecar — see [Concepts → Security fundamentals](../concepts/security-fundamentals.md).

## Reporting vulnerabilities

**Do not open a public issue for security vulnerabilities.** Email <security@legionforge.org>. We respond within 5 business days. After a fix is in place and users have had a chance to update, we publish a security advisory in the affected repo with the coordinated CVE if one was assigned.
