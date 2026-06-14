# Getting Started

This page walks through installing LegionForge, configuring the runtime, and running your first task.

!!! note "Pre-release"
    LegionForge is at **v0.7.1-alpha** and in UAT before the v0.8.0 public release. The source repository is being prepared for public availability. Installation instructions below describe the intended workflow.

## Prerequisites

| Component | Version | Why |
|---|---|---|
| **macOS** | 13+ (Apple Silicon recommended) | Primary target; the hardware profile system optimizes for M-series Macs |
| **Python** | 3.11 | Pinned via pyenv; required for the asyncio + LangGraph combo |
| **PostgreSQL** | 17 (Homebrew) | State layer, LangGraph checkpoints, pgvector RAG |
| **Ollama** | latest | Local LLM runtime |
| **Docker Desktop** | latest | Required for the Guardian sidecar |

Linux and Windows are supported but the hardware profiles are tuned for Apple Silicon.

## Install

```bash
git clone https://github.com/LegionForge/LegionForge.git
cd LegionForge

# Python 3.11 via pyenv
pyenv install 3.11.15
pyenv local 3.11.15

# Virtual environment
python3 -m venv venv
source venv/bin/activate

# Dependencies
pip install -r requirements.txt
```

## Configure

LegionForge reads a hardware profile at startup. The default profile is for a Mac Mini M4 with 16 GB of RAM:

```bash
export AGENT_HARDWARE_PROFILE=mac_m4_mini_16gb
```

Profiles live in `config/hardware_profiles/`. Switch via the env var.

Secrets live in the macOS Keychain — not in `.env`. The framework reads them at startup:

```bash
# Example: store a Tavily search API key
security add-generic-password -A -s legionforge_tavily_api_key -a api_key -w <your-key>
```

The `-A` flag is important — it makes the key readable by background processes, not just the app that created it.

## Bring up infrastructure

```bash
make check       # verifies drive, venv, models, config
make db-init     # one-time PostgreSQL + LangGraph + app table setup
make start       # full startup: drive check → Ollama → PostgreSQL → model warmup
```

## Run the smoke test

The smoke suite is fast (~21 s, no external services required):

```bash
make test-smoke
```

You should see **2247 passing** at the v0.7.1-alpha baseline.

## Send your first task

Start the gateway and the web UI:

```bash
make gateway-start
```

The gateway listens on `localhost:8080`. Open the web UI in a browser and submit a task. The response streams back over SSE.

To send a task from the command line:

```bash
curl -X POST http://localhost:8080/tasks \
  -H "Authorization: Bearer <your-key>" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the current time in UTC?"}'
```

## Next steps

- [Architecture](architecture.md) — understand the module layout and request flow
- [Security Model](security-model.md) — how validation, tool signing, and Guardian fit together
- [Gateway & API](gateway.md) — full API reference for the FastAPI gateway
- [Connectors](connectors.md) — wire LegionForge into Discord, Telegram, Slack, WhatsApp, or a webhook

## Troubleshooting

??? warning "`make start` fails at the database step"

    Make sure PostgreSQL is running and the `legionforge_admin` password is in your Keychain. Verify with:

    ```bash
    security find-generic-password -s postgres -a api_key -w
    ```

??? warning "Ollama models don't load"

    Models live at `/Volumes/MAC_MINI_1TB/ollama_models/` by default. If your storage path differs, set `OLLAMA_MODELS` in your environment before `make start`. Check the hardware profile for the expected model names.

??? warning "Guardian container won't start"

    Verify the `legionforge_guardian` PostgreSQL password is in Keychain:

    ```bash
    security find-generic-password -s legionforge_guardian -a api_key -w
    ```

    If it returns nothing, reset the role password and re-add it. Guardian also requires Docker Desktop running.
