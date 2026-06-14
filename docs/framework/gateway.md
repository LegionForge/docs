# Gateway & API

The gateway is a FastAPI application that exposes LegionForge to the outside world. It runs on `localhost:8080` by default and provides:

- Task submission with auth
- Server-Sent Events (SSE) streaming
- Web UI for interactive use
- Health, metrics, and status endpoints
- A2A (Agent-to-Agent) endpoints
- MCP (Model Context Protocol) endpoints

## Starting the gateway

```bash
make gateway-start
```

The Makefile target injects all required secrets from Keychain as env vars, so the gateway can read them regardless of SSH session ACLs.

## Authentication

Every request requires a Bearer token:

```bash
curl -H "Authorization: Bearer <your-key>" http://localhost:8080/health
```

Keys are hashed (Argon2) and stored in `gateway_users`. Creating a key:

```bash
make create-user USER=alice QUOTA_DAILY=100000
```

The CLI prints the unhashed key once at creation. Store it; it cannot be retrieved later.

## Submitting a task

```http
POST /tasks
Authorization: Bearer <your-key>
Content-Type: application/json

{
  "prompt": "Summarize the README at https://example.com/repo",
  "options": {
    "model": "llama3.1:8b",
    "token_budget": 8000,
    "tracing": false
  }
}
```

Response:

```json
{
  "task_id": "task_01HZ3...",
  "status": "queued",
  "stream_url": "/tasks/task_01HZ3.../stream"
}
```

## Streaming the response

Connect to the SSE stream:

```bash
curl -N -H "Authorization: Bearer <your-key>" \
  http://localhost:8080/tasks/task_01HZ3.../stream
```

You'll receive events like:

```
event: tool_call
data: {"tool": "web_fetch", "args": {"url": "https://example.com/repo"}}

event: tool_result
data: {"tool": "web_fetch", "result": "..."}

event: message
data: {"role": "assistant", "content": "The README describes..."}

event: done
data: {"status": "completed", "tokens": 4231, "cost_usd": 0.0}
```

## Endpoints

### Public

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness check. No auth. |
| `GET` | `/metrics` | Prometheus metrics. No auth (bind to localhost only). |
| `GET` | `/` | Web UI. |

### Authenticated

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/tasks` | Submit a task. |
| `GET` | `/tasks/{id}` | Get task status + result. |
| `GET` | `/tasks/{id}/stream` | SSE stream of task events. |
| `POST` | `/tasks/{id}/approve` | Approve a HITL gate. |
| `POST` | `/tasks/{id}/deny` | Deny a HITL gate. |
| `GET` | `/usage` | Per-user usage statistics. |

### A2A (Agent-to-Agent)

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/a2a/invoke` | Synchronous invocation from another agent. |
| `GET` | `/a2a/capabilities` | List capabilities this agent exposes. |

### MCP (Model Context Protocol)

LegionForge can be both an MCP **client** (consuming external MCP servers) and an MCP **server** (exposing its tools to external MCP clients). The server endpoints follow the MCP standard.

## Rate limiting

Every request is subject to:

1. **Per-key token quota** — daily token budget from `gateway_users.quota_daily`. Soft alert at 80%, hard cut at 100%.
2. **Per-provider rate limit** — pre-execution cost estimation in `rate_limiter.py` blocks the call if it would exceed the daily provider cap.
3. **Per-IP rate limit** — middleware drops repeated submissions from the same address.

Exceeded limits return `429 Too Many Requests` with a `Retry-After` header.

## Web UI

The web UI is the easiest way to send tasks interactively. It supports:

- Multi-modal image input (drag-and-drop)
- Session continuity (revisit and resume past tasks)
- HITL approval prompts
- Live cost meter
- Tool-call inspector

It's served from `/` when the gateway is running. The API key is stored in `localStorage` under `lf_api_key` and reused on subsequent visits.

## Trace IDs

Every request gets a UUID trace ID injected at the FastAPI middleware layer. It's surfaced as the `X-Request-ID` response header and threaded through to LangSmith if tracing is enabled. Useful for correlating logs across the gateway, worker, and Guardian.
