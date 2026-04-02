# Pattern: Process Groups + External Postgres

Single Fly app, multiple process groups, Postgres hosted outside Fly (Supabase, Neon, RDS, etc.).

---

## What This Looks Like

One app (`prod-myapp`) runs multiple processes defined in `fly.toml`. Postgres lives entirely outside Fly — either with a managed provider the client already has, or one chosen for features MPG/fly pg don't offer. Connection goes over the public internet (TLS enforced by default with all major providers).

---

## Postgres Setup

### Get your connection string

From your provider's dashboard. Always use the **connection pooler URL** where available:
- **Supabase**: use the pooler endpoint (port 6543, pgbouncer mode) not the direct connection
- **Neon**: use the pooled connection string, not the direct one
- **RDS / other**: set up pgbouncer separately or manage pool size at the app layer

### Set as a secret

```bash
fly secrets set DATABASE_URL="postgres://user:pass@host:5432/dbname?sslmode=require" --app prod-myapp
```

`sslmode=require` should always be present for external providers. Most providers enforce it but be explicit.

Do the same for staging:
```bash
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp
```

### Latency

External Postgres adds network latency vs `.internal`. Mitigate by:
- Choosing a provider region close to your Fly app region (`yyz` = Toronto → use `us-east-1` or a Canadian region where available)
- Keeping queries lean — you're paying per round-trip more than you are on internal Postgres
- Using a connection pooler to avoid handshake overhead per query

---

## Dockerfile Entrypoint

Process group commands are passed as `CMD` to your Dockerfile's `ENTRYPOINT` — they don't replace the entrypoint. Your entrypoint needs to handle being passed a command, otherwise process groups silently run whatever the default CMD is regardless of what you defined in `[processes]`.

Standard pattern:

```bash
#!/bin/bash
if [[ $# -gt 0 ]]; then
  exec "$@"
else
  exec /app/server  # default if no command passed
fi
```

```dockerfile
COPY entrypoint.sh /app/
ENTRYPOINT ["/app/entrypoint.sh"]
```

## fly.toml

```toml
app = "prod-myapp"
primary_region = "yyz"

[build]

[env]
  PORT = "3000"
  NODE_ENV = "production"

[processes]
  web = "bun run start:web"
  worker = "bun run start:worker"

[http_service]
  processes = ["web"]
  internal_port = 3000
  force_https = true
  auto_stop_machines = "off"
  auto_start_machines = true
  min_machines_running = 1

  [http_service.concurrency]
    type = "requests"
    hard_limit = 250
    soft_limit = 200

  [[http_service.checks]]
    grace_period = "10s"
    interval = "30s"
    method = "GET"
    path = "/health"
    protocol = "http"
    timeout = "5s"

# worker: internal only, autostop if lightly used
[[services]]
  processes = ["worker"]
  internal_port = 4000
  auto_stop_machines = "suspend"
  auto_start_machines = true
  min_machines_running = 0

[[vm]]
  processes = ["web"]
  memory = "512mb"
  cpu_kind = "shared"
  cpus = 1

[[vm]]
  processes = ["worker"]
  memory = "1gb"
  cpu_kind = "shared"
  cpus = 1
```

No Postgres-specific fly.toml config needed — the connection is entirely via `DATABASE_URL` secret.

---

## Secrets

```bash
fly secrets set DATABASE_URL="postgres://..." --app prod-myapp
fly secrets set ANTHROPIC_API_KEY="sk-..." --app prod-myapp
```

All secrets apply to all process groups. If `web` and `worker` need different db credentials (e.g. worker gets write access, web gets read-only), that's a signal to split into separate apps instead.

---

## Scaling Process Groups Independently

```bash
fly scale count 2 --process-group web --app prod-myapp
fly scale vm performance-2x --process-group worker --app prod-myapp
```

---

## Provider-Specific Notes

**Supabase**
- Use pooler endpoint (port 6543) for application connections — direct connection (port 5432) is for migrations only
- Supabase auth, storage, and realtime are available as side benefits if the project uses them
- Free tier is generous for staging; Pro tier ($25/mo) for prod

**Neon**
- Branching is useful for staging — create a branch from prod snapshot rather than a separate cluster
- Serverless autoscaling to zero means cold starts on first connection; disable autoscaling for prod if that's unacceptable
- Use pooled connection string; direct connections bypass the pooler

**RDS / Aurora**
- Highest latency from Fly since it's on a different cloud; cross-cloud egress costs add up
- If client is already on AWS and moving to Fly, acceptable transitional setup; long-term, migrate closer

---

## Common Pitfalls

- **Missing `sslmode=require`** — be explicit even if the provider enforces it; avoids surprises on connection string changes
- **Using direct connection instead of pooler** — burns through connection limits fast under any real load
- **Cross-region Postgres** — picking a provider region far from `yyz` will hurt; check available regions for each provider
- **Process groups are not co-located in one VM** — each group runs on its own Machine(s)
- **Deploys are atomic** — deploying `prod-myapp` redeploys both `web` and `worker` together
- **Secrets shared** — `DATABASE_URL` goes to all process groups, no scoping possible
