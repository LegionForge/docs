# LegionForge Documentation

Source for the **LegionForge documentation hub** served at [docs.legionforge.org](https://docs.legionforge.org/).

This is the central documentation site for the entire LegionForge ecosystem:

- The **framework** ([LegionForge/LegionForge](https://github.com/LegionForge/LegionForge))
- **Guardian** ([LegionForge/guardian](https://github.com/LegionForge/guardian))
- Tools: [llm-valet](https://github.com/LegionForge/llm-valet), [mcp-probe](https://github.com/LegionForge/mcp-probe), [headroom](https://github.com/LegionForge/headroom), [hermes-tool-test-suite](https://github.com/LegionForge/hermes-tool-test-suite), [dev-rig](https://github.com/LegionForge/dev-rig), [idn-analyzer](https://github.com/LegionForge/Intelligence-Delivery-Network-Request-Analyzer)
- Apps: [jeli](https://github.com/LegionForge/jeli), [ADHD-OS](https://github.com/LegionForge/ADHD-OS), [ConvoBox](https://github.com/LegionForge/convobox)
- Orchestration and research: [Briarios](https://github.com/LegionForge/Briarios), [Intelligence Delivery Network](https://github.com/LegionForge/Intelligence-Delivery-Network)

## Local development

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Then open <http://127.0.0.1:8000/>.

## Build

```bash
mkdocs build --strict
```

Output goes to `site/` (gitignored).

## Deploy

Pushes to `main` trigger `.github/workflows/deploy.yml`, which builds with MkDocs and deploys to GitHub Pages. The custom domain is set via the `CNAME` file in the repo root and the `docs.legionforge.org` DNS record at Namecheap.

## Editing pages

Every page has an "edit this page" pencil icon in the top-right that links straight to the source markdown on GitHub. Easiest way to make a small correction.

## License

Documentation is dual: prose under AGPL-3.0 (matching the framework), code examples under the license of their source project.
