# Guardian Architecture

## Component layout

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Agent framework (LegionForge / LangChain / ...)          │
│                                                             │
│   For every tool invocation:                                │
│      POST http://guardian:9766/check                        │
│      ↓                                                      │
│      ←  { allow: true } or { allow: false, reason: "..." } │
│                                                             │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP (port 9766)
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Guardian container                                        │
│                                                             │
│   FastAPI app                                               │
│   ├─ /check         — run all 7 checks                     │
│   ├─ /health        — liveness                              │
│   ├─ /metrics       — Prometheus                            │
│   ├─ /invalidate    — force rule cache refresh             │
│   └─ /canary        — synthetic check, for monitoring      │
│                                                             │
│   Rule cache (in-memory)                                    │
│      ↑ refreshed every 10s from PostgreSQL                  │
│                                                             │
└────────────────────────┬────────────────────────────────────┘
                         │ asyncpg
                         ↓
┌─────────────────────────────────────────────────────────────┐
│   PostgreSQL                                                │
│   ├─ tool_registry  (read-only for guardian role)          │
│   └─ threat_rules   (read + write for guardian role)       │
└─────────────────────────────────────────────────────────────┘
```

## Request lifecycle

1. The agent framework decides to call a tool.
2. It posts a JSON payload to `POST /check`:
   ```json
   {
     "tool_name": "shell_exec",
     "task_id": "task_01HZ3...",
     "user_id": 42,
     "capability_scope": ["read", "shell:safe"],
     "args": {"command": "ls /tmp"},
     "code_hash": "sha256:..."
   }
   ```
3. Guardian runs all 7 checks in order. The first to fail short-circuits.
4. Response:
   - `200 OK` `{"allow": true, "trace_id": "..."}` → proceed
   - `200 OK` `{"allow": false, "reason": "destructive_pattern: rm -rf detected", "check_name": "destructive_pattern_detection"}` → deny
   - `503` → halt (by policy, the framework does not fall back to running without Guardian)
5. Guardian writes a row to `threat_events` for every denial, asynchronously.

The check pipeline is **all-pass** semantics: a check that doesn't apply returns "pass". A check that **does** apply and fails returns "deny" with a reason.

## Rule storage

Adaptive rules live in `threat_rules`:

```sql
CREATE TABLE threat_rules (
    id BIGSERIAL PRIMARY KEY,
    rule_name TEXT NOT NULL UNIQUE,
    enabled BOOLEAN NOT NULL DEFAULT true,
    match_pattern TEXT NOT NULL,   -- regex or jsonpath
    match_field TEXT NOT NULL,     -- 'tool_name' | 'args' | 'code_hash' | ...
    action TEXT NOT NULL,          -- 'deny' | 'flag' | 'log'
    reason TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

Guardian fetches all enabled rules at startup and refreshes the cache every 10 seconds. New rules go live without restarting any agent process.

Adding a rule:

```sql
INSERT INTO threat_rules (rule_name, match_pattern, match_field, action, reason)
VALUES ('block_aws_metadata_endpoint',
        '169\.254\.169\.254',
        'args',
        'deny',
        'AWS metadata service IP is not allowed');
```

The next check within 10 seconds will pick it up.

## Why deterministic, not LLM

The reflex when designing a security layer in 2026 is to put an LLM in it — "have the model judge whether this call is safe." That's wrong, for three reasons:

1. **Latency.** A check that takes 200ms is unusable when an agent makes 30 tool calls per task. Determinism keeps the budget under 5ms.
2. **Cost.** Every tool call would be an LLM call. Compound costs.
3. **Adversarial robustness.** LLM-based checks can be prompt-injected by the very payloads they're inspecting.

Guardian sticks to regex, hash compare, signature verify, and lookups. They're crude. They're also predictable and auditable.

## Health and observability

| Endpoint | Purpose |
|---|---|
| `GET /health` | Liveness. Returns 200 if Guardian is up and rule cache is non-empty. |
| `GET /metrics` | Prometheus metrics — check duration, deny rate per check, cache age. |
| `POST /invalidate-cache` | Force a rule cache reload immediately. Useful after pushing new rules and not wanting to wait 10s. |
| `GET /canary` | Run a synthetic `/check` against a known-bad payload. Returns 200 if Guardian correctly denied. Used for liveness probes. |

The canary endpoint is the most important monitoring tool: if Guardian is alive but silently failing to deny anything (e.g., rule cache empty due to DB auth failure), the canary catches it where `/health` wouldn't.

## What happens if Guardian is down

By policy, the framework **halts execution**. There is no fall-back-to-no-Guardian path. The reasoning is:

- The cost of halting a task is recoverable (operator restarts Guardian, task resumes from LangGraph checkpoint).
- The cost of executing tool calls without Guardian's checks is potentially unbounded.

If the framework can't reach Guardian, it logs `GUARDIAN_UNREACHABLE` and refuses tool calls.

## Drop-in usage for other frameworks

Guardian is HTTP. Any agent framework can adopt it with a small wrapper:

```python
import httpx

GUARDIAN_URL = "http://localhost:9766"

async def guarded_tool_call(tool_name, args, task_context):
    response = await httpx.AsyncClient().post(
        f"{GUARDIAN_URL}/check",
        json={
            "tool_name": tool_name,
            "task_id": task_context.task_id,
            "user_id": task_context.user_id,
            "capability_scope": task_context.scope,
            "args": args,
            "code_hash": get_tool_hash(tool_name),
        },
    )
    decision = response.json()
    if not decision["allow"]:
        raise PermissionError(decision["reason"])
    return await actually_call_tool(tool_name, args)
```

That's the entire integration. Plug Guardian into LangChain, AutoGen, CrewAI, or a custom agent loop the same way.
