# Pattern: Separate Apps + Fly Managed Postgres (MPG)

Two or more Fly apps with independent fly.toml files, communicating via private network, backed by Fly Managed Postgres.

---

## What This Looks Like

- `prod-myapp-web` — public-facing, handles HTTP traffic
- `prod-myapp-api` — internal-only or separately deployed API
- MPG cluster — provisioned in the Fly dashboard, lives on the org's default private network

All apps are in the same org and share a private network (6PN). Web talks to API via `.internal`. Both connect to MPG via the connection string from the dashboard.

---

## Postgres Setup

### Provision MPG

Via the Fly dashboard. Choose a plan:

| Plan | CPU | RAM | $/mo (+ $0.28/GB storage) |
|------|-----|-----|---------------------------|
| Basic | Shared-2x | 1GB | $38 |
| Starter | Shared-2x | 2GB | $72 |
| Launch | Performance-2x | 8GB | $282 |

Storage is **provisioned, not consumed** — size it with headroom, you pay for the allocation.

**Check region availability first** — MPG is not yet in all regions.

### Connect each app

`fly pg attach` does **not** work with MPG. Set `DATABASE_URL` manually on each app. You can use the same connection string on both, or create separate database users per app for credential isolation (do the latter in the MPG dashboard if needed).

```bash
fly secrets set DATABASE_URL="postgres://user:pass@host.flycast:5432/dbname" --app prod-myapp-web
fly secrets set DATABASE_URL="postgres://user:pass@host.flycast:5432/dbname" --app prod-myapp-api
```

For staging:
```bash
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp-web
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp-api
```

### Connection pooling

MPG includes PgBouncer. The connection string from the dashboard points to the pooler by default. Don't bypass it — connecting directly to the Postgres port skips pooling.

With two apps connecting to the same MPG cluster, the pooler handles the aggregate connection load automatically.

---

## Inter-App Networking

Web to API via `.internal`:
```
http://prod-myapp-api.internal:4000
```

Direct, no Fly Proxy. Keep the API internal-only — no public `[[services.ports]]` unless it genuinely needs public exposure.

---

## fly.toml — Web App

```toml
app = "prod-myapp-web"
primary_region = "yyz"

[build]

[env]
  PORT = "3000"
  NODE_ENV = "production"
  API_URL = "http://prod-myapp-api.internal:4000"

[http_service]
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

[[vm]]
  memory = "512mb"
  cpu_kind = "shared"
  cpus = 1
```

## fly.toml — API App (internal-only)

```toml
app = "prod-myapp-api"
primary_region = "yyz"

[build]

[env]
  PORT = "4000"
  HOST = "::"  # 6PN is IPv6 — bind to :: or .internal traffic won't reach the app
  NODE_ENV = "production"

[[vm]]
  memory = "1gb"
  cpu_kind = "shared"
  cpus = 1
```

---

## Secrets

Set independently per app — full isolation:

```bash
fly secrets set DATABASE_URL="postgres://..." --app prod-myapp-web
fly secrets set DATABASE_URL="postgres://..." --app prod-myapp-api
fly secrets set ANTHROPIC_API_KEY="sk-..." --app prod-myapp-web
```

---

## Deployments

Independent per app:

```bash
fly deploy --config fly.prod.toml --app prod-myapp-web
fly deploy --config fly.prod.toml --app prod-myapp-api
```

---

## Scaling

```bash
fly scale count 3 --app prod-myapp-web
fly scale vm performance-2x --app prod-myapp-api
```

---

## MPG-Specific Notes

**What Fly handles:**
- HA and automatic failover
- Automated backups and PITR
- PgBouncer connection pooling (shared across all connecting apps)
- Monitoring, metrics, alerting
- 24/7 support

**What you still own:**
- Plan sizing — pick based on expected load from all apps combined, not just one
- Provisioned storage — monitor actual vs allocated, resize before ceiling
- Extension constraints — only pgvector and PostGIS beyond default extensions

**6PN isolation caveat**: MPG lives on the org's default private network. Apps on custom 6PN networks (e.g. `fly apps create --network prod-network`) cannot resolve the MPG `.flycast` hostname — connections will fail silently or timeout. If you need environment isolation, use separate orgs instead of custom networks.

---

## Common Pitfalls

- **`fly pg attach` doesn't work** — set `DATABASE_URL` manually as a secret on each app

- **Storage is provisioned** — don't under-allocate; resize before you hit the ceiling
- **Don't bypass PgBouncer** — the MPG connection string points to the pooler; use it
- **Region availability** — verify MPG is available in your target region before planning the architecture
