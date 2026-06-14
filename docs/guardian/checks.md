# Guardian Checks

Guardian runs 7 deterministic checks on every tool invocation, in this order. The first failing check short-circuits the pipeline.

## 1. Tool revocation list

**What it catches:** known-bad tool IDs.

When a tool is found to be malicious, vulnerable, or deprecated, its ID is added to the revocation list. The list is a small table queried in O(1) memory; the entire list lives in Guardian's in-memory cache.

```sql
CREATE TABLE revoked_tools (
    tool_id TEXT PRIMARY KEY,
    revoked_at TIMESTAMPTZ DEFAULT now(),
    reason TEXT NOT NULL
);
```

**Decision:** if `tool_id IN revoked_tools` → deny with `reason = "tool_revoked: <reason>"`.

## 2. Hash validation

**What it catches:** supply-chain tool tampering.

Every registered tool has a SHA-256 of its loaded code in `tool_registry`. At invocation time, Guardian re-hashes the live code and compares.

A mismatch means the tool's source changed since it was registered — possibly due to a dependency upgrade replacing a function, a malicious package update, or local file tampering.

**Decision:** if `hash(live_code) != registry.expected_hash` → deny with `reason = "hash_mismatch"`. Threat event `TOOL_HASH_MISMATCH` is logged.

## 3. Capability boundary

**What it catches:** capability creep — an agent invoking a tool outside its task's authorized scope.

Every task carries a capability scope: an array of strings like `["read", "fetch:web", "summarize"]`. Every tool declares its required capability: `shell_exec` requires `shell:safe` or `shell:full`; `db_write` requires `db:write`.

**Decision:** if `tool.required_capability NOT IN task.capability_scope` → deny with `reason = "capability_boundary: missing <capability>"`.

This is the layer that stops the "give me capability X just for this one thing" sliding-scale failure mode. Scope is set at task creation and never widens during execution.

## 4. Destructive pattern detection

**What it catches:** dangerous arguments — usually the result of an LLM splicing user-controlled text into a tool argument.

The check is a list of regex patterns applied to JSON-serialized args:

| Pattern | Why |
|---|---|
| `rm\s+-rf\s+/` | Recursive root delete |
| `(DROP|TRUNCATE)\s+TABLE` | Destructive SQL |
| `:(){:|:&};:` | Fork bomb |
| `curl.+(\|sh|\|bash)` | Pipe-to-shell |
| `169\.254\.169\.254` | Cloud metadata service IP |
| `chmod\s+777` | World-writable permission |

The pattern set is extensible via `threat_rules` (action = `deny`, match_field = `args`).

**Decision:** if any pattern matches → deny with `reason = "destructive_pattern: <pattern>"`.

## 5. Sequence contracts

**What it catches:** out-of-order tool calls that violate stated contracts.

Some tool sequences have implicit ordering. Examples:

- `delete_file` should be preceded by `read_file` (so the agent has acknowledged what it's deleting)
- `db_commit` should be preceded by `db_begin`
- `email_send` should be preceded by `email_draft` (so HITL can review)

Sequence contracts are configured in the `sequence_contracts` table:

```sql
CREATE TABLE sequence_contracts (
    contract_name TEXT PRIMARY KEY,
    requires_prior TEXT NOT NULL,    -- tool name that must precede
    triggers_for TEXT NOT NULL,      -- tool name that triggers the check
    within_steps INTEGER NOT NULL DEFAULT 5
);
```

**Decision:** if `triggers_for` is invoked and `requires_prior` was not invoked within the last `within_steps` → deny with `reason = "sequence_contract: <contract_name>"`.

## 6. Ed25519 signature verification

**What it catches:** unauthorized tool registration.

Tools aren't just hashed — they're signed. The Ed25519 private key lives in macOS Keychain (`legionforge_tool_signer`) and is injected as `TOOL_SIGNING_PRIVATE_KEY` env var only at signing time. The public key is in Guardian's config.

Every row in `tool_registry` has a `signature` column. Guardian verifies `signature(hash) == registry.signature` using the public key.

**Decision:** if signature verification fails → deny with `reason = "signature_invalid"`.

This catches the case where someone with PostgreSQL write access inserts a malicious tool registration. They can write the row, but they can't produce a valid signature without the private key.

## 7. Adaptive threat rules

**What it catches:** novel patterns discovered post-deployment.

This is the extension point. The `threat_rules` table holds operator-defined rules that fire on:

- Tool name
- Argument content
- Code hash
- Combinations

Rules are hot-reloaded every 10 seconds, so a new rule goes live without redeploying anything.

```sql
INSERT INTO threat_rules (rule_name, match_pattern, match_field, action, reason)
VALUES ('block_external_drive_writes',
        '/Volumes/(?!MAC_MINI_1TB)',
        'args',
        'deny',
        'Writes to non-canonical external drives are blocked');
```

**Decision:** if any enabled rule matches and `action = deny` → deny with `reason = "adaptive_rule: <rule_name>"`.

Rules with `action = flag` are logged as `threat_events` but the call is allowed. Useful for measuring how often a candidate rule would fire before turning it into a `deny`.

## Performance characteristics

| Check | Typical latency |
|---|---|
| 1. Revocation list | ~0.05ms (in-memory lookup) |
| 2. Hash validation | ~0.5ms (SHA-256 of typical tool code) |
| 3. Capability boundary | ~0.05ms (set membership) |
| 4. Destructive patterns | ~0.5ms (regex pipeline) |
| 5. Sequence contracts | ~0.3ms (history query) |
| 6. Signature verification | ~1ms (Ed25519 verify) |
| 7. Adaptive rules | ~1-3ms (regex pipeline, depends on rule count) |
| **Total** | **~3-5ms in practice** |

The total budget is small enough that Guardian sits in the hot path of every tool call without measurable user-visible impact.
