# Guardian Deployment

Guardian is shipped as a Docker container. Recommended deployment is via the LegionForge framework's `make guardian-start` target, which handles Keychain secrets and the database role correctly. For other frameworks, deploy with docker-compose directly.

## With the LegionForge framework

```bash
make guardian-start
```

The Make target:

1. `docker rm -f legionforge-guardian` — ensures no stale env vars from a prior run
2. Reads `TASK_TOKEN_SECRET` from Keychain (`legionforge_task_tokens`)
3. Reads `POSTGRES_PASSWORD` from Keychain (`legionforge_guardian` — **not** the admin password)
4. Explicitly exports `POSTGRES_USER=legionforge_guardian` to override `.env`
5. Starts the container on the default bridge network (not `legionforge-net` — see [Networking](#networking) below)

## Standalone with docker-compose

If you're using Guardian with a non-LegionForge framework, here's a minimal `docker-compose.yml`:

```yaml
version: "3.9"

services:
  guardian:
    image: legionforge-guardian:latest
    ports:
      - "9766:9766"
    environment:
      POSTGRES_HOST: host.docker.internal
      POSTGRES_PORT: "5432"
      POSTGRES_DB: legionforge
      POSTGRES_USER: legionforge_guardian
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      TASK_TOKEN_SECRET: ${TASK_TOKEN_SECRET}
      TOOL_SIGNING_PUBLIC_KEY: ${TOOL_SIGNING_PUBLIC_KEY}
      RULE_REFRESH_INTERVAL_SECONDS: "10"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9766/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

## Database setup

Guardian needs a PostgreSQL role with the **minimum** privileges to read tool registrations and read/write threat data. Don't give Guardian the admin role.

```sql
-- As the admin role:
CREATE ROLE legionforge_guardian WITH LOGIN PASSWORD '<secret>';

GRANT CONNECT ON DATABASE legionforge TO legionforge_guardian;
GRANT USAGE ON SCHEMA public TO legionforge_guardian;

GRANT SELECT ON tool_registry, revoked_tools, sequence_contracts
  TO legionforge_guardian;

GRANT SELECT, INSERT, UPDATE ON threat_rules, threat_events
  TO legionforge_guardian;

GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public
  TO legionforge_guardian;
```

This means even if Guardian is compromised, the blast radius is contained: it can read tool metadata, read/write rules, and append threat events. It cannot read user data, modify tasks, or grant itself more privileges.

## Environment variables

| Variable | Purpose |
|---|---|
| `POSTGRES_HOST` | PostgreSQL host. Use `host.docker.internal` on Mac/Windows; the bridge IP or service name on Linux. |
| `POSTGRES_PORT` | Default `5432`. |
| `POSTGRES_DB` | The application database. |
| `POSTGRES_USER` | **Must be the restricted role** (e.g. `legionforge_guardian`). Never use the admin role. |
| `POSTGRES_PASSWORD` | Password for the role above. |
| `TASK_TOKEN_SECRET` | JWT signing secret for task tokens. Shared with the framework. |
| `TOOL_SIGNING_PUBLIC_KEY` | Ed25519 public key (base64). Guardian verifies signatures with this. |
| `RULE_REFRESH_INTERVAL_SECONDS` | How often to reload the rule cache. Default 10. |

## Networking

Guardian binds `9766/tcp`. The framework connects over HTTP. Two networking patterns:

- **Loopback (recommended for single-host):** Guardian and framework both on the host's default bridge; framework uses `http://localhost:9766`.
- **Container-to-container:** if both are on the same custom bridge network, framework uses `http://guardian:9766`.

!!! warning "Don't put Guardian on `legionforge-net`"
    The framework's internal `legionforge-net` Docker network is isolated from host routing. Guardian must remain on the default bridge so the framework on the host can reach it. The LegionForge Makefile sets this up automatically.

## Health probes

Production deployments should configure two probes:

- **Liveness:** `GET /health` — returns 200 when the process is up and the rule cache is loaded.
- **Behavioral:** `GET /canary` — sends a synthetic known-bad payload through `/check` and confirms the deny path actually fires. Catches "Guardian is up but its rule cache is empty due to DB auth failure" — a state where `/health` returns 200 but no rule actually fires.

Run the canary probe at least as often as the liveness probe.

## Metrics

`GET /metrics` exposes Prometheus metrics:

| Metric | Type | Labels |
|---|---|---|
| `guardian_check_duration_seconds` | histogram | `check_name`, `decision` |
| `guardian_decisions_total` | counter | `decision`, `check_name` |
| `guardian_rule_cache_size` | gauge | — |
| `guardian_rule_cache_age_seconds` | gauge | — |
| `guardian_db_query_duration_seconds` | histogram | `query` |

A useful alert: `guardian_rule_cache_size == 0` for more than 30 seconds. That's the failure mode the canary probe catches; this alert catches the same state from the metric side.

## Upgrading

Guardian follows semantic versioning. The image is published as `legionforge-guardian:<version>` and `:latest` on Docker Hub. To upgrade:

```bash
docker pull legionforge-guardian:latest
make guardian-start   # picks up the new image
```

Database migrations, if any, run automatically on first start of a new version. The migration sequence is forward-only — there is no automatic downgrade.
