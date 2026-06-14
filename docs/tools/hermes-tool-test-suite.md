# hermes-tool-test-suite

**pytest harness for validating Hermes AI agent tool-calling reliability.**

[:fontawesome-brands-github: github.com/LegionForge/hermes-tool-test-suite](https://github.com/LegionForge/hermes-tool-test-suite){ .md-button .md-button--primary }

## What it does

The hermes-tool-test-suite is a pytest harness that validates AI agent tool-calling reliability across providers. It's used to certify that a given model + prompt-config combination actually **executes** tool calls end-to-end, not just generates text that looks like a tool call.

The default validation set covers 10 scenarios across:

| Category | Scenarios |
|---|---|
| Terminal | shell exec, cwd handling |
| File ops | read, write, edit, list |
| Code execution | inline Python, subprocess Python |
| Web tools | fetch, search |

Each scenario has a deterministic expected output. The model's tool-call sequence is run and the resulting state (filesystem, stdout, etc.) is compared.

## Status

Active. Public.

## Why this exists

A model that generates a `tool_calls` block in its API response hasn't actually proven it can drive tools. Real failures observed in practice:

- Model generates `tool_calls` but the framework's tool dispatcher trips on a tool name that's been mangled (e.g., underscore stripping)
- Model returns text describing a tool call instead of using the structured `tool_calls` field
- Model loops on the same call without integrating the result

A test harness that runs the entire loop catches all three.

## Use cases

- **Pre-deployment certification** — before shipping a new model to production, run the suite. If 9/10 pass, you know what the failure mode is and whether it's blocking.
- **Model comparison** — A/B testing tool-call reliability between providers (Ollama llama3.1 vs qwen2.5 vs cloud Claude vs OpenAI).
- **Regression detection** — after upgrading a framework version, re-run the suite to confirm no regression.

## Integration with LegionForge

LegionForge's primary inference path runs through `llm_factory` which is tested by this suite. The framework's CI runs a subset of this harness on every merge.
