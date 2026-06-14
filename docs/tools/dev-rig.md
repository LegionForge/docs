# dev-rig

**Shared Python CI workflows, pre-commit config, and audit harness for LegionForge projects.**

[:fontawesome-brands-github: github.com/LegionForge/dev-rig](https://github.com/LegionForge/dev-rig){ .md-button .md-button--primary }

## What it does

dev-rig is the **shared CI/CD substrate** used across every LegionForge repo. It provides:

- **Reusable GitHub Actions workflows** — lint, test, SAST, dependency audit, secrets scan, SBOM generation, container scan (Trivy)
- **Pre-commit configuration** — Black, isort, ruff, mypy, bandit, detect-secrets
- **Audit harness** — Bandit + pip-audit + URI scrubbing, runnable as a single make target

The goal is that every project under the LegionForge org has the same security/quality baseline without copy-pasting workflow files between repos.

## Status

Active. Public. Internal tooling — most useful if you're contributing to a LegionForge project or want to use the same baseline in your own.

## Using it in a LegionForge project

In a project repo's `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  lint:
    uses: LegionForge/dev-rig/.github/workflows/lint.yml@main

  test:
    uses: LegionForge/dev-rig/.github/workflows/test.yml@main
    with:
      python-version: "3.11"

  sast:
    uses: LegionForge/dev-rig/.github/workflows/sast.yml@main

  audit:
    uses: LegionForge/dev-rig/.github/workflows/audit.yml@main

  secrets:
    uses: LegionForge/dev-rig/.github/workflows/secrets.yml@main

  sbom:
    uses: LegionForge/dev-rig/.github/workflows/sbom.yml@main
```

That's the entire CI config — every workflow is sourced from dev-rig. Updating dev-rig updates the CI across all projects that reference `@main`.

## Pre-commit

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/LegionForge/dev-rig
    rev: v0.1.0
    hooks:
      - id: legionforge-baseline
```

The `legionforge-baseline` hook runs Black, ruff, isort, bandit, and detect-secrets. It's pinned to a tag, not `main`, so projects only adopt new rules deliberately.

## When to use it outside LegionForge

If you maintain multiple Python repos and want a consistent security baseline, dev-rig is a good template. The workflows are MIT-licensed and the configuration is intentionally vanilla — they don't assume LegionForge-specific structure.
