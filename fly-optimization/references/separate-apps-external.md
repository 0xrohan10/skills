# Pattern: Separate Apps + External Postgres

Two or more Fly apps with independent fly.toml files, communicating via private network, Postgres hosted outside Fly.

---

## What This Looks Like

- `prod-myapp-web` — public-facing
- `prod-myapp-api` — internal-only or separately deployed
- Postgres — Supabase, Neon, RDS, or any external provider

Apps communicate internally via 6PN. Both connect to external Postgres over the public internet (TLS). This is the most flexible topology — nothing ties you to Fly's database options.

---

## Postgres Setup

### Get your connection string

From your provider's dashboard. Always use the **connection pooler URL** where available:
- **Supabase**: pooler endpoint port 6543 (pgbouncer mode) for app connections; port 5432 direct for migrations only
- **Neon**: pooled connection string, not the direct one
- **RDS / other**: configure pgbouncer separately or manage pool size at the app layer

### Set as a secret on each app

```bash
fly secrets set DATABASE_URL="postgres://user:pass@host:5432/dbname?sslmode=require" --app prod-myapp-web
fly secrets set DATABASE_URL="postgres://user:pass@host:5432/dbname?sslmode=require" --app prod-myapp-api
```

You can use the same credentials on both apps, or create separate database users per app for credential isolation — depends on how tightly you want to scope access.

`sslmode=require` should always be explicit, even if the provider enforces it.

For staging:
```bash
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp-web
fly secrets set DATABASE_URL="postgres://..." --app staging-myapp-api
```

### Latency

External Postgres adds round-trip latency vs `.internal`. Minimize it:
- Pick a provider region close to your Fly app region (`yyz` = Toronto → prefer `us-east-1`, `ca-central-1`, or a Canadian region where available)
- Use the connection pooler — avoids per-query handshake overhead
- Keep queries lean; you're paying per round-trip more than with internal Postgres

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
  NODE_ENV = "production"

[[vm]]
  memory = "1gb"
  cpu_kind = "shared"
  cpus = 1
```

No Postgres-specific fly.toml config needed — the connection is entirely via `DATABASE_URL` secret.

---

## Secrets

Full isolation per app:

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

## Provider-Specific Notes

**Supabase**
- Port 6543 (pooler) for app connections; port 5432 for migrations only
- Auth, storage, realtime available if the project uses them
- Free tier fine for staging; Pro ($25/mo) for prod
- Canadian data residency: not natively available — data lives in AWS us-east-1 by default; check compliance requirements before using for healthcare clients

**Neon**
- Branching is great for staging — branch from a prod snapshot rather than running a separate cluster
- Autoscaling to zero causes cold starts; disable for prod if latency on first connection matters
- Use pooled connection string always

**RDS / Aurora**
- Highest latency from Fly since it's cross-cloud; egress costs add up at scale
- Acceptable as a transitional setup if client is already on AWS; plan to evaluate migration long-term
- RDS Proxy can help with connection management if you're hitting limits

---

## Common Pitfalls

- **Missing `sslmode=require`** — be explicit even if provider enforces it
- **Using direct connection instead of pooler** — burns connection limits fast under real load
- **Cross-region Postgres** — provider region far from `yyz` will hurt p99 latency noticeably

- **Canadian data residency** — external providers often default to US regions; verify for healthcare clients
