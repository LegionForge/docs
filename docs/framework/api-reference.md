# API Reference

!!! info "Coming soon"
    The auto-generated Python API reference is planned for a follow-up. It will be built with [`mkdocstrings`](https://mkdocstrings.github.io/) and will introspect the framework's source on every doc build.

    Until it's wired up, refer to module docstrings in the source repo directly:

    - [`config/settings.py`](https://github.com/LegionForge/LegionForge/blob/main/config/settings.py)
    - [`src/base_graph.py`](https://github.com/LegionForge/LegionForge/blob/main/src/base_graph.py)
    - [`src/security/core.py`](https://github.com/LegionForge/LegionForge/blob/main/src/security/core.py)
    - [`src/security/guardian.py`](https://github.com/LegionForge/LegionForge/blob/main/src/security/guardian.py)
    - [`src/safeguards.py`](https://github.com/LegionForge/LegionForge/blob/main/src/safeguards.py)
    - [`src/database.py`](https://github.com/LegionForge/LegionForge/blob/main/src/database.py)
    - [`src/llm_factory.py`](https://github.com/LegionForge/LegionForge/blob/main/src/llm_factory.py)
    - [`src/rate_limiter.py`](https://github.com/LegionForge/LegionForge/blob/main/src/rate_limiter.py)
    - [`src/gateway/app.py`](https://github.com/LegionForge/LegionForge/blob/main/src/gateway/app.py)

## Planned approach

When the framework is published to PyPI, the docs build will `pip install legionforge` and run mkdocstrings on the installed package. Until then, the choice is between:

- **git submodule** — vendor the framework source into this repo at build time
- **CI git clone** — clone fresh in the GitHub Action that builds the docs

The CI clone is the most likely choice: it doesn't pollute this repo's history and always builds against `main` of the framework.

## What it will cover

- All public classes in `src/`
- The Pydantic settings model from `config/settings.py`
- Tool decorators and the `SecureToolNode` API
- Threat event types as a Python enum
- The connector base class for writing new bridges

## What it won't cover

- Internal `_` -prefixed helpers
- Test scaffolding
- Migration scripts
