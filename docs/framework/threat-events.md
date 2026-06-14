# Threat Events

Security-relevant events are written to the `threat_events` table in PostgreSQL. The table is append-only and feeds the Phase 4 Threat Analyst agent that analyzes patterns across deployments.

## Schema

```sql
CREATE TABLE threat_events (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT now(),
    event_type TEXT NOT NULL,
    task_id TEXT,
    user_id BIGINT,
    severity TEXT NOT NULL,       -- 'info' | 'warning' | 'critical'
    payload JSONB NOT NULL,
    source_module TEXT NOT NULL
);
```

## Event types

| Event type | Severity | What triggers it |
|---|---|---|
| `INJECTION_DETECTED` | critical | Tier-1 prompt-injection pattern matched in `sanitize_input()`. The input was rejected. |
| `TOOL_HASH_MISMATCH` | critical | A loaded tool's code hash did not match its stored hash in `tool_registry`. Invocation blocked. |
| `MODEL_INTEGRITY_MISMATCH` | critical | A model's weights or metadata didn't match the expected fingerprint at load time. |
| `PREFLIGHT_BUDGET_EXCEEDED` | warning | A pre-execution cost estimate would have exceeded the per-task token budget. The call was blocked before being made. |
| `PII_REDACTED` | info | `sanitize_output()` removed PII from text being logged or returned. |
| `LOOP_DETECTED` | warning | The action-history hash showed the same tool-call signature 3 times within a 5-step window. Task halted. |
| `STEP_LIMIT_REACHED` | warning | LangGraph hit the hard recursion limit. Task halted. |
| `TOKEN_BUDGET_EXCEEDED` | warning | Cumulative token usage for the task crossed 100% of its budget. Task force-ended. |
| `TOOL_ARG_INJECTION` | critical | Suspicious patterns in tool call arguments produced by the LLM. Common attack: agent passing user-controlled content into a shell tool unsanitized. |
| `TOOL_RESULT_INJECTION` | critical | Tool output contained patterns indicative of injection back toward the model (e.g., a fetched web page containing prompt-injection payloads). |
| `GUARDIAN_DENIED` | warning | Guardian denied a tool invocation. The payload contains which check failed. |
| `GUARDIAN_UNREACHABLE` | critical | Framework could not reach Guardian on its expected port. By policy, this halts execution; the framework will not fall back to running without Guardian. |
| `HITL_DENIED` | info | A human-in-the-loop approval gate was denied by the operator. |
| `RATE_LIMIT_TRIGGERED` | warning | A per-user, per-provider, or per-IP rate limit fired. |
| `KEY_NOT_FOUND` | critical | A Keychain lookup at startup returned empty for a required secret. |

## Payload conventions

The `payload` JSONB column carries event-specific detail. By convention:

- For `INJECTION_DETECTED`, `payload.matched_patterns` is an array of pattern IDs.
- For `TOOL_HASH_MISMATCH`, `payload.tool_name`, `payload.expected_hash`, `payload.actual_hash`.
- For `LOOP_DETECTED`, `payload.action_signature` (the MD5 hash that repeated) and `payload.window` (the actual history slice).
- For `TOKEN_BUDGET_EXCEEDED`, `payload.budget`, `payload.actual`, `payload.breakdown` by tool.
- For `GUARDIAN_DENIED`, `payload.check_name` (which of the 7 checks failed), `payload.tool_name`, `payload.reason`.

Never log raw user prompts or secrets in payloads. The sanitization that runs at the trust boundary is enforced before writing.

## Querying

Common queries:

```sql
-- Event counts by type, last 7 days
SELECT event_type, COUNT(*)
FROM threat_events
WHERE timestamp > now() - interval '7 days'
GROUP BY event_type
ORDER BY COUNT(*) DESC;

-- All critical events from a specific user
SELECT timestamp, event_type, payload
FROM threat_events
WHERE user_id = 42
  AND severity = 'critical'
ORDER BY timestamp DESC
LIMIT 50;

-- Recent Guardian denials with reasons
SELECT timestamp, payload->>'tool_name' AS tool, payload->>'check_name' AS check, payload->>'reason' AS reason
FROM threat_events
WHERE event_type = 'GUARDIAN_DENIED'
  AND timestamp > now() - interval '24 hours'
ORDER BY timestamp DESC;
```

## Retention

By default, `threat_events` is retained for 90 days (set in the hardware profile as `audit_log_retention_days`). The same retention window applies to `audit_log`. Older rows are pruned by a nightly maintenance job.

For compliance scenarios that require longer retention, the recommended approach is to export to cold storage rather than extending the live retention window — `threat_events` is hot, indexed, and read frequently.
