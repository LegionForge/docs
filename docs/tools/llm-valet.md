# llm-valet

**Cross-platform LLM lifecycle manager** — auto-pause and resume Ollama based on system resource pressure and gaming detection.

[:fontawesome-brands-github: github.com/LegionForge/llm-valet](https://github.com/LegionForge/llm-valet){ .md-button .md-button--primary }

## The problem it solves

Running a local LLM is expensive in RAM. A 7B-parameter model in 4-bit quantization is ~4.5 GB; an 8B at q4 is ~5.5 GB. On a 16 GB machine, keeping Ollama warm means you have ~10 GB for everything else — a Chromium tab tree, a game, a video editor, Slack — and the moment the system swaps, the LLM gets evicted unpredictably and re-loads on the next request (~30 s).

llm-valet sits in the middle. It watches:

- **Memory pressure** — if the system is paging or close to it, pause Ollama models cleanly.
- **GPU pressure** — if another GPU-hungry process spins up (a game, a render), pause the model.
- **Gaming detection** — recognized game processes are a strong signal that the user wants their hardware back.

When the pressure subsides, llm-valet warms the model back up automatically.

## Status

Active. Initial public version. Cross-platform (macOS, Linux, Windows) but tested most heavily on Apple Silicon.

## When to use it

- You run Ollama locally and use the same machine for other workloads.
- You don't want LLM memory pressure to thrash the rest of your system.
- You want the model to come back automatically when there's room — not on the next slow request.

## When not to use it

- Dedicated inference box (no other workloads). Just let Ollama hold memory.
- Cloud-only LLM use. There's nothing to pause.

## Integration with LegionForge

LegionForge can run alongside llm-valet without coordination — both work independently. For tighter integration (e.g., have the framework respect llm-valet pause state instead of making a request that triggers an unwanted reload), see the [llm-valet README](https://github.com/LegionForge/llm-valet#legionforge-integration).
