# Configuration

LegionForge's runtime configuration lives in a single Pydantic singleton loaded from a YAML hardware profile. There is no `.env` file for production secrets — those go in the macOS Keychain.

## The settings singleton

Everything reads config the same way:

```python
from config.settings import settings

print(settings.primary_model_name)         # "llama3.1:8b"
print(settings.token_budget_per_task)      # 32000
print(settings.guardian_url)               # "http://localhost:9766"
```

`settings` is a Pydantic v2 `BaseSettings` instance. It's hot — every import returns the same object.

## Hardware profiles

The active profile is selected by an environment variable:

```bash
export AGENT_HARDWARE_PROFILE=mac_m4_mini_16gb
```

Profiles live in `config/hardware_profiles/`. The default is `mac_m4_mini_16gb.yaml`:

```yaml
# Memory limits
total_ram_gb: 16
ollama_memory_budget_gb: 10
gateway_memory_budget_gb: 2

# Model selection
primary_model_name: llama3.1:8b
router_model_name: qwen2.5:3b
embedding_model_name: mxbai-embed-large

# Safeguard thresholds
token_budget_per_task: 32000
loop_detection_window: 5
loop_detection_threshold: 3

# Paths
ollama_models_dir: /Volumes/MAC_MINI_1TB/ollama_models/
audit_log_retention_days: 90
```

To run with a different profile, drop a new YAML file into `config/hardware_profiles/` and export the env var:

```bash
export AGENT_HARDWARE_PROFILE=mac_m2_air_8gb
```

## Secrets

Secrets are read from the macOS Keychain at startup. The framework reads them via the `security` CLI rather than the `keyring` Python library, because background processes (the gateway, the Guardian container) can fail to find keys via `keyring` due to ACL isolation.

### Storing a secret

```bash
security add-generic-password \
  -A \                              # any-app ACL — readable by background processes
  -s legionforge_tavily_api_key \   # service name
  -a api_key \                      # account name (convention)
  -w <your-key>                     # secret value
```

The `-A` flag is critical. Without it, background processes can't read the key and you get silent fallbacks.

### Reading a secret manually

```bash
security find-generic-password -s legionforge_tavily_api_key -a api_key -w
```

### Secrets the framework expects

| Service | Account | Purpose |
|---|---|---|
| `postgres` | `api_key` | PostgreSQL admin password (`legionforge_admin` role) |
| `legionforge_guardian` | `api_key` | PostgreSQL password for the Guardian Docker container's restricted role |
| `legionforge_task_tokens` | `api_key` | JWT signing secret for task tokens |
| `legionforge_tool_signer` | `api_key` | Ed25519 private key for tool signing |
| `legionforge_tavily_api_key` | `api_key` | Tavily search API key (primary) |
| `legionforge_brave_api_key` | `api_key` | Brave search API key (fallback) |

Provider keys (OpenAI, Anthropic, InceptionLabs) follow the same pattern: `legionforge_<provider>_api_key`.

## Per-task overrides

Some settings are per-task: token budget, model selection, tracing on/off. These are passed in the task submission payload to the gateway and override the profile defaults for that task only:

```json
{
  "prompt": "Summarize this PDF",
  "options": {
    "model": "claude-sonnet-4-6",
    "token_budget": 8000,
    "tracing": true
  }
}
```

## Switching providers

The `llm_factory` reads provider configuration from the profile. To swap from local Ollama to cloud Anthropic for a session:

```bash
export AGENT_DEFAULT_PROVIDER=anthropic
make gateway-start
```

Cloud fallback can also be configured to trigger automatically when local models are insufficient (e.g., when the prompt exceeds the local model's context window).
