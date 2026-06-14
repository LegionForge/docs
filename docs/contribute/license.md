# License

LegionForge uses a **dual-license model** for the main framework and a permissive license for Guardian. Documentation is also dual-licensed.

## Main framework — AGPL-3.0 + commercial

The [LegionForge framework](https://github.com/LegionForge/LegionForge) is licensed under **GNU AGPL-3.0** with a **Section 7(b) attribution requirement** and a **commercial license option**.

### What AGPL-3.0 means in practice

- You can use, modify, and distribute LegionForge freely for any purpose.
- If you run a modified version as a **network service**, you must offer source code of your modified version to users of that service.
- You must include attribution per Section 7(b).

This is the right license for:

- Personal use
- Open-source projects
- Internal-only use within a company (no public network service)
- Research

### When you need the commercial license

You need the **commercial license** if you want to:

- Embed LegionForge in a proprietary product without releasing your modifications under AGPL
- Run LegionForge as a paid network service without exposing your source modifications
- Avoid the Section 7(b) attribution requirement

Commercial licenses are available from <jp@legionforge.org>. See `COMMERCIAL_LICENSE.md` in the framework repo for terms.

## Guardian — MIT

[Guardian](https://github.com/LegionForge/guardian) is licensed under the **MIT License**, independent of the framework.

The deliberate choice: Guardian is positioned as a primitive that any agent framework can adopt. AGPL would discourage adoption by frameworks that ship under permissive licenses. MIT removes that friction.

You can:

- Embed Guardian in proprietary products
- Distribute modified versions without source disclosure
- Use it with any framework, AGPL or not

## Tools and apps

Most tools and apps under the LegionForge org are individually licensed. Check each repo's `LICENSE` file. The defaults:

| Type | Default license |
|---|---|
| Internal/CI tooling (dev-rig, etc.) | MIT |
| Standalone libraries (Guardian, mcp-probe, etc.) | MIT |
| Framework integrations | AGPL-3.0 + commercial (matching the framework) |
| Apps (Jeli, ADHD-OS) | AGPL-3.0 |

## Documentation

The prose on this site (these documentation pages) is licensed under **AGPL-3.0**, matching the framework. Code snippets embedded in the docs follow the license of their source project.

## Why dual-license

The dual-license model lets LegionForge stay open-source while supporting a sustainable funding path. Personal users and open-source projects get it for free under AGPL. Companies that build on it commercially contribute back through the commercial license. The framework owner makes the call about which terms apply to a given user.

## Contact

| Reason | Contact |
|---|---|
| Commercial license inquiries | <jp@legionforge.org> |
| Security disclosures | <security@legionforge.org> |
| General questions | GitHub Issues in the relevant repo |
