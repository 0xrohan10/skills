# Pattern: Process Groups + Fly Postgres (Unmanaged)

Single Fly app, multiple process groups, Fly-managed-but-not-really Postgres cluster.

---

## What This Looks Like

One app (`prod-myapp`) runs multiple processes defined in `fly.toml`. A separate Fly app (`prod-myapp-db`) runs the Postgres cluster. Both live in the same org, same private network.

---

## Postgres Setup

### Create the cluster

For prod, always use HA (3-node):
```bash
fly pg create --name prod-myapp-db --region yyz --initial-cluster-size 3
```

For staging, single node is fine:
```bash
fly pg create --name staging-myapp-db --region yyz --initial-cluster-size 1
```

### Attach to the app

`fly pg attach` creates a `DATABASE_URL` secret on your app automatically. With process groups, this secret is visible to **all groups** — there's no way to scope it to one process only.

```bash
fly pg attach --app prod-myapp --postgres-app prod-myapp-db
```

This sets `DATABASE_URL` as a secret on `prod-myapp`. Do the same for staging:
```bash
fly pg attach --app staging-myapp --postgres-app staging-myapp-db
```

### Disable scale-to-zero on prod

Fly Postgres dev clusters scale to zero after 1hr idle. Explicitly prevent this on prod:
```bash
fly scale count 1 --app prod-myapp-db
```

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

`DATABASE_URL` is set automatically by `fly pg attach`. Any additional secrets:
```bash
fly secrets set ANTHROPIC_API_KEY="sk-..." --app prod-myapp
```

All secrets apply to all process groups. If `web` and `worker` need different credentials, that's a signal to split into separate apps instead.

---

## Scaling Process Groups Independently

```bash
# scale web horizontally
fly scale count 2 --process-group web --app prod-myapp

# scale worker vertically
fly scale vm performance-2x --process-group worker --app prod-myapp
```

---

## Operational Notes (Fly pg)

**You own these:**
- **Disk**: Monitor volume usage. If it fills, you're SSHing in. Set up Prometheus alerting on `pg_database_size_bytes`.
- **OOM**: If Postgres runs out of memory it crashes and you bring it back. Size your db VM accordingly.
- **No PgBouncer**: Fly pg doesn't bundle a connection pooler. Manage connection pool size at the app layer (e.g. Drizzle pool config). If you're running many app instances and hitting connection limits, deploy a separate pgbouncer app.
- **Backups**: Daily snapshots included. PITR is not. Know your RPO before committing.
- **Upgrades**: Major version upgrades are manual. `fly pg` tooling helps but it's not zero-touch.

**HA failover**: The 3-node HA cluster handles primary failure automatically via Patroni. Replica lag during heavy writes is something to watch.

---

## Common Pitfalls

- **Process groups are not co-located in one VM** — each group runs on its own Machine(s)
- **Deploys are atomic** — deploying `prod-myapp` redeploys both `web` and `worker` together; no independent deploy per group
- **Secrets shared** — `DATABASE_URL` goes to all process groups, no scoping possible
- **Staging db scales to zero** — fine for staging, but first query after idle will cold-start Postgres; acceptable tradeoff
