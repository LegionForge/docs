# Contributing

LegionForge is a multi-repo ecosystem. The contribution workflow is roughly the same across all of them; this page covers the conventions that apply everywhere, with project-specific notes where they differ.

## Before you start

1. **Open an issue first** for non-trivial work. Discuss the approach before writing code. Saves both sides from wasted effort.
2. **Read the project's CLAUDE.md** if it has one — it captures conventions and footguns that aren't in the README.
3. **Run the test suite locally** so you know the baseline before you touch anything.

## Workflow

```
main ← feature/xxx (or fix/xxx, refactor/xxx, docs/xxx, security/xxx)
```

- Branch off `main`
- One concern per commit / PR. Don't bundle UI changes, agent logic changes, and test changes in a single commit.
- Conventional commits: `feat:`, `fix:`, `chore:`, `security:`, `docs:`, `refactor:`, `test:`
- Pre-commit hook runs Black, ruff, and bandit. The hook will reformat; re-stage after the first run if your file gets touched.

## Tests

Every project under LegionForge runs CI via [dev-rig](../tools/dev-rig.md). The minimum gate for a PR:

- `make test` (or the equivalent) passes
- `make security-audit` (or equivalent) passes
- No new findings from Bandit or pip-audit at `medium` or higher severity

For the **framework**, the additional gate is `make ci` — it runs smoke + testlab + ui suites *plus* the audit, and the smoke test count must never decrease (current baseline: 2247).

## Style

- Black-formatted (project Python version is 3.11). The `make format` target auto-fixes.
- Type hints on public functions. Internal helpers don't have to.
- Don't add abstractions for hypothetical futures. Three similar lines is better than a premature base class.
- Don't add error handling for cases that can't happen. Trust internal data; validate at trust boundaries.
- Avoid Co-Authored-By: trailers from squash-merge if you don't want them on the public commit history (this has bitten the project before).

## Pull request checklist

A good PR template:

```markdown
## Summary
What changed and why, in 1-3 bullets.

## Test plan
- [ ] Tests added / updated for the new behavior
- [ ] `make ci` passes locally
- [ ] Smoke baseline preserved (or, if intentionally raised, noted here)

## Threat model impact
Any change to security-relevant code paths (sanitize_input, sanitize_output,
Guardian checks, tool signing, capability scoping) — note the impact here
or write "no impact".

## Migrations
List any PostgreSQL schema changes or required ops steps.
```

## Security disclosures

**Do not open a public issue for security vulnerabilities.** Email <security@legionforge.org>. We'll respond within 5 business days. After a fix is in place and users have had a chance to update, we'll publish a security advisory in the affected repo.

## Code of conduct

Be respectful. Take feedback in good faith. Assume positive intent. The contributor experience matters as much as the code.

## CLA

Contributing code requires accepting the Contributor License Agreement. The CLA is presented automatically when you open your first PR. The CLA grants the project the rights necessary to dual-license under AGPL-3.0 and the commercial license — it does not transfer copyright.
