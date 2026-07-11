# Security

The Security section is for people evaluating LegionForge as user-owned AI infrastructure. The Framework and Guardian sections cover *how* LegionForge enforces checks. This section covers *why* the model is built around custody, auditability, and human authority.

## Pages

<div class="grid cards" markdown>

-   :material-shield-alert: **[Threat Model](threat-model.md)**

    STRIDE applied to agent systems. The kill-chain view: attacker goal → vector → which LegionForge layer catches it (or honestly, doesn't).

-   :material-compare: **[Differentiators](differentiators.md)**

    Side-by-side: what changes when the user is the root of trust instead of a vendor account or an unguarded agent loop.

-   :material-magnify-scan: **[OpenClaw incident analysis](openclaw-incident.md)**

    What the Jan 2026 OpenClaw incident exposed about fast-moving agent ecosystems, and which architectural patterns LegionForge addresses.

</div>

## Where to go from here

- For the *technical* security model — what runs where, what the deterministic checks look like — see [Framework → Security Model](../framework/security-model.md).
- For Guardian's specific behavior — the 7 checks, the rule storage, the canary endpoint — see [Guardian → Architecture](../guardian/architecture.md).
- For an introduction to the *thinking* behind the model — the LLM-is-not-trustworthy thesis, why deterministic, why sidecar — see [Concepts → Security fundamentals](../concepts/security-fundamentals.md).

## Reporting vulnerabilities

**Do not open a public issue for security vulnerabilities.** Email <security@legionforge.org>. We respond within 5 business days. After a fix is in place and users have had a chance to update, we publish a security advisory in the affected repo with the coordinated CVE if one was assigned.
