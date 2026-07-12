# Project Security Inventory

This page is the working map for LegionForge repository coverage: what each
repo is for, what kind of security automation is visible, and what still needs
to be wired into the shared baseline.

The goal is not to make every repo look equally mature. The goal is to make
coverage explicit so gaps are easy to prioritize.

## Coverage levels

| Level | Meaning |
|---|---|
| **Baseline CI** | The repo has automated lint/test/security workflows or equivalent checks on GitHub. |
| **Local audit** | The repo can be checked with `LegionForge-dev-rig/scripts/audit.sh`, including static/non-Python repos. |
| **Needs wiring** | The repo is public or operationally important, but visible workflow coverage is absent or incomplete. |
| **Review scope** | The repo is research, media, or early-stage work where the first task is deciding the right baseline. |

## Public repos

| Repo | Role | Primary stack | Visible automation | Current note | Next question |
|---|---|---|---|---|---|
| [LegionForge](https://github.com/LegionForge/LegionForge) | Core sovereign agent framework | Python | CI, smoke tests, PostgreSQL test workflow | Core runtime repo; should remain a first-class security target. | Are all reusable dev-rig jobs required on PRs? |
| [guardian](https://github.com/LegionForge/guardian) | Deterministic agent security sidecar | Python | CI, tests, DAST, OSS audit, publish workflow | High-security repo with broad workflow coverage. | Is the local checkout remote aligned with the public lowercase `guardian` repo? |
| [jeli](https://github.com/LegionForge/jeli) | Sovereign personal memory governance | Python | CI, workflow lint | Memory custody project; needs the strongest docs-to-tests traceability. | Are export, redaction, conflict resolution, and chain verification all covered by CI? |
| [convobox](https://github.com/LegionForge/convobox) | Local voice frontend for coding agents | Python | No workflow files visible in local checkout | Safety-critical because voice commands can trigger agent actions. | Should dev-rig CI become required before adapters stabilize? |
| [dev-rig](https://github.com/LegionForge/dev-rig) | Shared audit/CI baseline | Python, shell, GitHub Actions | Audit, CI, lint, SAST, SBOM, secrets, supply-chain workflows | Source of truth for common checks. Local audit now supports static repos. | Should consuming repos pin tags or track `@main`? |
| [docs](https://github.com/LegionForge/docs) | Documentation hub | MkDocs | Pages deploy workflow | Public security claims live here. | Should docs PRs require link/build checks and static audit? |
| [legionforge.github.io](https://github.com/LegionForge/legionforge.github.io) | Public website | Jekyll/static site | No workflow files visible in local checkout | Published via GitHub Pages from the repo. Static audit passes through dev-rig. | Should Pages deployment and link checks be explicit workflows? |
| [mcp-probe](https://github.com/LegionForge/mcp-probe) | MCP configuration/connectivity advisor | TypeScript | Audit, CI, SAST, secrets workflows | Network-adjacent diagnostic tool; good candidate for strict supply-chain checks. | Are npm lockfile audits and Semgrep required on PRs? |
| [llm-valet](https://github.com/LegionForge/llm-valet) | Local LLM lifecycle/resource manager | Python | CI, publish workflow | Operational daemon touching local resource policy. | Does CI include dependency audit, SAST, and secrets scan? |
| [headroom](https://github.com/LegionForge/headroom) | System stability monitor | Rust | Not reviewed locally | Resource-monitoring tool. | Should Rust-specific checks include `cargo test`, `cargo audit`, and `cargo deny`? |
| [hermes-tool-test-suite](https://github.com/LegionForge/hermes-tool-test-suite) | Hermes tool-call validation harness | Python | No workflow files visible in local checkout | Test harness repo; useful for regression confidence. | Should this consume dev-rig CI even if it primarily tests another project? |
| [ADHD-OS](https://github.com/LegionForge/ADHD-OS) | Personal assistant framework | Early-stage | No workflow files visible in local checkout | Product direction appears early-stage. | What privacy/security claims are safe to make publicly today? |
| [Briarios](https://github.com/LegionForge/Briarios) | Agentic workstream orchestration research | Early-stage | No workflow files visible in local checkout | Directional orchestration repo. | Is this research-only or planned runtime infrastructure? |
| [Intelligence-Delivery-Network](https://github.com/LegionForge/Intelligence-Delivery-Network) | LLM routing/research concept | Research | No workflow files visible in local checkout | Research repo with requirements file locally. | Should public docs label this as research until runtime hardening exists? |
| [Intelligence-Delivery-Network-Request-Analyzer](https://github.com/LegionForge/Intelligence-Delivery-Network-Request-Analyzer) | Lightweight prompt/request profiler | C / Python-adjacent files locally | No workflow files visible in local checkout | Analysis tool; may need language-specific checks. | What build/test command should define a passing audit? |
| [media](https://github.com/LegionForge/media) | Brand/media assets | Shell/static assets | Pages publish, asset validation workflows | Non-runtime repo, but still receives secret/static checks. | Should asset provenance and licensing be documented? |

## Private repos

| Repo | Role | Current note | Next question |
|---|---|---|---|
| `legionforge-orchestration` | Internal project orchestration | Private operational repo. | Which checks are mandatory before automations can touch public repos? |
| `screenwright` | AI UI documentation pipeline | Private Python repo. | Does it need browser/credential isolation rules documented? |
| `firmwright` | Firmware dev/test/QA/OTA harness | Private Python repo. | Does OTA or device access require a stricter release gate? |

## Baseline questions to answer for each repo

- **What is the deployment target?** Website, docs, package, daemon, local CLI, research artifact, or private operational tool.
- **What can it touch?** Filesystem, network, microphone, database, device firmware, credentials, memory store, or public publishing surface.
- **What is the required CI baseline?** Lint, tests, dependency audit, SAST, secrets scan, SBOM, container scan, language-specific checks.
- **What is the local audit command?** Prefer `LegionForge-dev-rig/scripts/audit.sh /path/to/repo`; document any repo-specific setup.
- **What must block release?** Failing tests, dependency vulnerabilities, secrets, unsigned artifacts, unchecked destructive actions, or missing docs.
- **How does a user leave?** Export, backup, restore, migration, deletion, and ownership boundaries should be documented for user-state projects.

## Current priorities

1. Wire dev-rig CI into public repos that have no visible workflow coverage.
2. Require secret scanning and dependency/SBOM checks for docs, website, and app repos.
3. Add language-specific coverage where dev-rig is not enough: Rust for `headroom`, TypeScript/npm for `mcp-probe`, and C/build checks for the request analyzer.
4. Document deployment paths for `docs` and `legionforge.github.io`, including the branch and workflow that publish production.
5. Keep this inventory updated whenever a repo is created, archived, renamed, or promoted from research to runtime infrastructure.
