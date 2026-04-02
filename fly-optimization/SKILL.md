---
name: fly-optimization
description: >
  Opinionated best practices for deploying and configuring apps on Fly.io.
  Use this skill whenever a Fly.io deployment is being created, reviewed, or modified —
  including fly.toml configuration, process topology decisions, postgres setup, secrets,
  networking between apps, or autostop/autostart settings. Trigger on any mention of
  fly.io, fly.toml, fly deploy, fly pg, fly mpg, flycast, .internal networking, or
  process groups in a deployment context.
---

# Fly.io Optimization

Opinionated best practices for consistent, production-ready Fly.io deployments.

Start here to determine the correct architecture, then load the appropriate reference doc for the full pattern.

---

## Step 1: Determine Topology

**Use `[processes]` in a single app when:**
- Same codebase and Docker image
- Same deploy cadence (one deploy = both services redeploy, and that's fine)
- Shared secrets are acceptable across all processes

**Use separate Fly apps when:**
- Different codebases / repos
- Need independent deploy cadences
- Need secret isolation between services
- One service is public, another is internal-only and owned separately

---

## Step 2: Determine Postgres

**Fly MPG (Managed Postgres)** — when ops overhead needs to be zero, client has no custom extension requirements, budget supports it (~$38+/mo base)

**Fly pg (unmanaged)** — when budget-conscious, need custom extensions (TimescaleDB etc), or comfortable owning db operations

**External provider (Supabase, Neon, etc.)** — when client already has a provider, needs features neither MPG nor fly pg offer, or Postgres needs to live outside Fly entirely

---

## Step 3: Load the Reference Doc

Based on the above, load the relevant file from `references/`:

| Topology | Postgres | Reference doc |
|----------|----------|---------------|
| Process groups | Fly pg | `references/process-groups-fly-pg.md` |
| Process groups | MPG | `references/process-groups-mpg.md` |
| Process groups | External | `references/process-groups-external.md` |
| Separate apps | Fly pg | `references/separate-apps-fly-pg.md` |
| Separate apps | MPG | `references/separate-apps-mpg.md` |
| Separate apps | External | `references/separate-apps-external.md` |

---

## Shared Conventions (apply to all patterns)

### Naming
- Format: `{env}-{appname}` — e.g. `prod-myapp`, `staging-myapp`
- Separate Fly app per environment, same org
- Separate `fly.toml` per environment
- Deploy: `fly deploy --config fly.prod.toml --app prod-myapp`

### Secrets
- Always `fly secrets set` — no `.env` files, no committed credentials
- Set independently per app and environment
- With process groups, secrets are shared across all groups — if isolation is needed, use separate apps instead

### Autostop
- **Default: off.** Production apps keep machines running
- **Exception:** lightly-used background workers (e.g. image processing) use `auto_stop_machines = "suspend"`
- Prefer `"suspend"` over `"stop"` — faster resume, same cost benefit at low traffic

### Static Assets

Don't run a separate file server process for static assets. Fly can serve them directly from the proxy — it pulls them out of your container at deploy time and serves from the edge:

```toml
[[statics]]
  guest_path = "/app/public"
  url_prefix = "/public"
```

Faster than serving from your app, and one less process to manage.

### Metrics

Fly includes a fully managed Prometheus + Grafana stack at fly-metrics.net. Built-in metrics (proxy, VM, OOM detection) are automatic. Custom metrics require a `/metrics` endpoint and a `[metrics]` block in fly.toml. See `references/metrics.md`.

### Logging

Write to stdout. Fly captures it automatically. See `references/logging.md` for how the stack works and how to access logs.


All public-facing services expose `GET /health` returning `200 OK`. Lightweight — no db queries unless gating on db connectivity is intentional.

```toml
[[http_service.checks]]
  grace_period = "10s"
  interval = "30s"
  method = "GET"
  path = "/health"
  protocol = "http"
  timeout = "5s"
```

### Process Group Lifecycle

**Every process group must have either `[http_service]` or `[[services]]`.** A `[checks]` block alone only monitors — it does NOT manage the machine's lifecycle. Without a service definition, a process group's machine:
- Has no `auto_start_machines` (won't restart after crashes)
- Has no `min_machines_running` (may stay stopped after deploys)
- Gets created during deploy but Fly won't keep it alive

Pattern for internal-only processes (API servers, workers accessed via `.internal`):

```toml
[[services]]
  processes = ["api"]
  internal_port = 8081
  protocol = "tcp"
  auto_stop_machines = "off"
  auto_start_machines = true
  min_machines_running = 1

  [[services.http_checks]]
    interval = "30s"
    timeout = "5s"
    grace_period = "10s"
    method = "GET"
    path = "/health"
```

**`[checks]` ≠ `[[services]]`.** Checks tell you something is wrong. Services keep things running.

### Dockerfile (Bun / Node monorepos)

**Don't use `--production` with Bun workspace monorepos.** `bun install --production` strips devDependencies from ALL workspace packages, not just the root. Workspace packages commonly miscategorize runtime deps as devDeps, causing silent startup failures in production.

**Always use `--ignore-scripts` in the production stage.** The `prepare` lifecycle script (typically `husky`) runs during install. In production Docker images, husky and other dev tooling aren't available. Skip it:

```dockerfile
# Runner stage
COPY package.json bun.lock ./
COPY packages ./packages
COPY apps ./apps
RUN bun install --frozen-lockfile --ignore-scripts
```

The same applies to npm/pnpm — `npm ci --ignore-scripts` is standard for production Docker builds.

### Networking
- All apps in the same org share a private IPv6 network (6PN)
- Internal traffic uses `.internal` DNS: `http://prod-myapi.internal:3000`
- **Your app must bind to `::` (all interfaces), not `127.0.0.1` or `0.0.0.0`** — 6PN is IPv6-only, so binding to an IPv4 address makes the service unreachable via `.internal`
- Never route internal traffic through public URLs
- No public `[[services.ports]]` on internal-only apps — absence of public ports = not internet-reachable, still 6PN-reachable

