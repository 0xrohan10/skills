# Pattern: Process Groups + Fly Managed Postgres (MPG)

Single Fly app, multiple process groups, fully managed Postgres via Fly MPG.

---

## What This Looks Like

One app (`prod-myapp`) runs multiple processes defined in `fly.toml`. MPG provides the Postgres cluster — provisioned separately via the Fly dashboard or CLI, lives on the org's default private network.

---

## Postgres Setup

### Provision MPG

MPG is provisioned via the Fly dashboard or CLI. Choose a plan and region close to your app:

| Plan | CPU | RAM | $/mo (+ $0.28/GB storage) |
|------|-----|-----|---------------------------|
| Basic | Shared-2x | 1GB | $38 |
| Starter | Shared-2x | 2GB | $72 |
| Launch | Performance-2x | 8GB | $282 |

Storage is **provisioned, not consumed** — you pay for what you allocate, not what you use.

**Check region availability before committing** — MPG is not in all regions yet.

### Connect to the app

`fly pg attach` does **not** work with MPG. Get the connection string from the Fly dashboard and set it manually:

```bash
fly secrets set DATABASE_URL="postgres://user:pass@host.flycast:5432/dbname" --app prod-myapp
```

Do the same for staging (separate MPG cluster or separate database on the same cluster):
```bash
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp
```

### Connection pooling

MPG includes PgBouncer — no need to manage pooling yourself. The connection string from the dashboard points to the pooler by default. Don't bypass it by connecting to the raw Postgres port.

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

---

## Secrets

```bash
fly secrets set DATABASE_URL="postgres://..." --app prod-myapp
fly secrets set ANTHROPIC_API_KEY="sk-..." --app prod-myapp
```

All secrets apply to all process groups. If `web` and `worker` need different credentials, that's a signal to split into separate apps instead.

---

## Scaling Process Groups Independently

```bash
fly scale count 2 --process-group web --app prod-myapp
fly scale vm performance-2x --process-group worker --app prod-myapp
```

---

## MPG-Specific Notes

**What Fly handles for you:**
- HA and automatic failover
- Automated backups and PITR
- PgBouncer connection pooling
- Monitoring, metrics, alerting
- 24/7 support response

**What you still own:**
- Choosing the right plan size for your workload
- Provisioned storage sizing — don't under-provision or you'll need to resize
- Extension compatibility — only pgvector and PostGIS available beyond default Postgres extensions; no custom extensions

**6PN isolation caveat**: MPG lives on the org's default private network. Apps on custom 6PN networks (e.g. `fly apps create --network prod-network`) cannot resolve the MPG `.flycast` hostname — connections will fail silently or timeout. If you need environment isolation, use separate orgs instead of custom networks.

---

## Common Pitfalls

- **`fly pg attach` doesn't work** — MPG connection string must be set as a secret manually
- **Process groups are not co-located in one VM** — each group runs on its own Machine(s)
- **Deploys are atomic** — deploying `prod-myapp` redeploys both `web` and `worker` together
- **Secrets shared** — `DATABASE_URL` goes to all process groups, no scoping possible
- **Storage is provisioned** — monitor actual usage vs provisioned; resize before you hit the ceiling
- **Don't bypass PgBouncer** — the MPG connection string points to the pooler; connecting directly to Postgres port skips it
