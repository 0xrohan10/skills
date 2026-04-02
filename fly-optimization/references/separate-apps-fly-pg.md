# Pattern: Separate Apps + Fly Postgres (Unmanaged)

Two or more Fly apps with independent fly.toml files, communicating via private network, sharing a Fly Postgres cluster.

---

## What This Looks Like

- `prod-myapp-web` — public-facing, handles HTTP traffic
- `prod-myapp-api` — internal-only or separately deployed API
- `prod-myapp-db` — Fly Postgres cluster, internal-only

All three live in the same org, same private network (6PN). Web talks to API via `.internal`. Both talk to Postgres via `.internal`.

---

## Postgres Setup

### Create the cluster

For prod, always use HA (3-node):
```bash
fly pg create --name prod-myapp-db --region yyz --initial-cluster-size 3
```

For staging, single node:
```bash
fly pg create --name staging-myapp-db --region yyz --initial-cluster-size 1
```

### Attach to each app separately

Unlike process groups where one attach sets the secret on the whole app, with separate apps you attach to each independently. This gives you the option to use different database users per app if you want credential isolation.

```bash
fly pg attach --app prod-myapp-web --postgres-app prod-myapp-db
fly pg attach --app prod-myapp-api --postgres-app prod-myapp-db
```

Each attach creates a `DATABASE_URL` secret on that specific app, scoped to a separate Postgres user. The databases can be the same or different — `fly pg attach` creates a new database by default but you can point both at the same one.

For staging:
```bash
fly pg attach --app staging-myapp-web --postgres-app staging-myapp-db
fly pg attach --app staging-myapp-api --postgres-app staging-myapp-db
```

### Disable scale-to-zero on prod

```bash
fly scale count 1 --app prod-myapp-db
```

---

## Inter-App Networking

Web to API communication uses `.internal` DNS:

```
http://prod-myapp-api.internal:3000
```

This is direct — no Fly Proxy involved. Keep the API internal-only — no public `[[services.ports]]` in its fly.toml unless it genuinely needs to be publicly reachable.

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

# No [http_service] with public ports — internal only
# If you need Fly Proxy features, add Flycast instead

[[vm]]
  memory = "1gb"
  cpu_kind = "shared"
  cpus = 1
```

---

## Secrets

Set independently per app:

```bash
# web app
fly secrets set ANTHROPIC_API_KEY="sk-..." --app prod-myapp-web
# DATABASE_URL already set by fly pg attach

# api app
fly secrets set SOME_API_SECRET="..." --app prod-myapp-api
# DATABASE_URL already set by fly pg attach
```

Secret isolation is a first-class benefit of this pattern — each app only gets the secrets it needs.

---

## Deployments

Each app deploys independently:

```bash
fly deploy --config fly.prod.toml --app prod-myapp-web
fly deploy --config fly.prod.toml --app prod-myapp-api
```

This is the main advantage over process groups — you can deploy the API without touching web, and vice versa.

---

## Scaling

Scale each app independently:

```bash
fly scale count 3 --app prod-myapp-web
fly scale vm performance-2x --app prod-myapp-api
```

---

## Operational Notes (Fly pg)

**You own these:**
- **Disk**: Monitor volume usage. Alert on `pg_database_size_bytes` via Prometheus.
- **OOM**: Size the db VM for your workload. If Postgres OOMs it crashes.
- **No PgBouncer**: Manage connection pool size at the app layer. Rule of thumb: `pool_size = (max_connections - 5) / total_instances_across_all_apps`. Default `max_connections` is 100 — with 2 apps × 3 instances each, that's ~15 per instance. Monitor with `SELECT count(*) FROM pg_stat_activity`. If you're consistently above 80% of `max_connections`, deploy a shared pgbouncer app that both route through.
- **Backups**: Daily snapshots included. PITR is not.
- **Upgrades**: Manual. `fly pg` tooling helps but it's not zero-touch.

---

## Common Pitfalls


- **Separate attach = separate db users** — by default `fly pg attach` creates a separate Postgres user per app; if you want both apps connecting as the same user, set `DATABASE_URL` manually instead of using attach
- **Staging db scales to zero** — first query cold-starts Postgres; fine for staging
- **Connection count** — two apps × N instances each × pool size = more connections than a single-app setup; watch this on smaller Postgres VM sizes
