# Separate Apps

Use separate Fly Apps for independent images, deploys, secrets, owners, scaling, or network exposure. Keep each environment in explicit apps and configs; deploy each app independently.

## Direct private service

Use `.internal` for direct Machine-to-Machine 6PN traffic when the private service stays running. `.internal` returns only started Machines, bypasses Fly Proxy, and cannot wake a stopped Machine. Bind the private listener to `fly-local-6pn` (or its actual 6PN address), not localhost.

Public web app:

```toml
app = "prod-example-web"
primary_region = "yyz"

[env]
  API_URL = "http://prod-example-api.internal:4000" # MEASURE: private API listen port
  HOST = "0.0.0.0"
  NODE_ENV = "production"
  PORT = "3000" # MEASURE: application listen port

[http_service]
  internal_port = 3000 # MEASURE: application listen port
  force_https = true
  auto_stop_machines = "off"
  auto_start_machines = false

  [http_service.concurrency]
    type = "requests"
    soft_limit = 200 # MEASURE: sustained per-Machine concurrency
    hard_limit = 250 # MEASURE: safe overload ceiling

  [[http_service.checks]]
    grace_period = "10s" # MEASURE: worst healthy startup time
    interval = "30s" # MEASURE: detection objective and check cost
    method = "GET"
    path = "/health"
    timeout = "5s" # MEASURE: healthy endpoint latency ceiling

[[vm]]
  cpu_kind = "shared"
  cpus = 1 # MEASURE: load test and CPU telemetry
  memory = "512mb" # MEASURE: peak RSS plus headroom
```

Always-on private API:

```toml
app = "prod-example-api"
primary_region = "yyz"

[env]
  HOST = "fly-local-6pn"
  HEALTH_HOST = "0.0.0.0"
  HEALTH_PORT = "4001" # MEASURE: dedicated top-level check listener
  NODE_ENV = "production"
  PORT = "4000" # MEASURE: private API listen port

[[restart]]
  policy = "always"

[checks]
  [checks.api]
    port = 4001 # MEASURE: dedicated top-level check listener
    type = "http"
    method = "get"
    path = "/health"
    grace_period = "10s" # MEASURE: worst healthy startup time
    interval = "30s" # MEASURE: detection objective and check cost
    timeout = "5s" # MEASURE: healthy endpoint latency ceiling

[[vm]]
  cpu_kind = "shared"
  cpus = 1 # MEASURE: load test and CPU telemetry
  memory = "1gb" # MEASURE: peak RSS plus headroom
```

The private app has no Fly Proxy service and no public ports. Its `always` restart policy, not the check, keeps it running. Top-level checks require a listener on `0.0.0.0`, so this example uses a dedicated health port while application traffic binds to `fly-local-6pn`.

## Proxy-wakeable private service

Use Flycast when a private HTTP service must stop or suspend and wake on requests. Configure a service, bind to `0.0.0.0:<internal_port>`, set `force_https = false`, allocate a private Flycast IPv6 address, remove public IPs, and make callers use `http://<app>.flycast:<port>`.

```toml
app = "prod-example-worker"
primary_region = "yyz"

[env]
  HOST = "0.0.0.0"
  PORT = "4000" # MEASURE: application listen port

[http_service]
  internal_port = 4000 # MEASURE: application listen port
  force_https = false
  auto_stop_machines = "suspend"
  auto_start_machines = true
  min_machines_running = 0 # MEASURE: cold-path latency tolerance

  [http_service.concurrency]
    type = "requests"
    soft_limit = 20 # MEASURE: safe simultaneous jobs per Machine

  [[http_service.checks]]
    grace_period = "15s" # MEASURE: cold-start or resume readiness
    interval = "30s" # MEASURE: detection objective and check cost
    method = "GET"
    path = "/health"
    timeout = "5s" # MEASURE: healthy endpoint latency ceiling

[[vm]]
  cpu_kind = "shared"
  cpus = 1 # MEASURE: job throughput target
  memory = "1gb" # MEASURE: peak job RSS plus headroom
```

Before deploy, verify privacy rather than assuming it:

```bash
fly ips allocate-v6 --private --app prod-example-worker
fly ips list --app prod-example-worker
```

Release every public address shown by `fly ips list`. Suspend may resume stale network connections or cold-start instead; clients and database pools must reconnect on failure. A queue-polling worker has no incoming proxy request to wake it, so keep it running or redesign dispatch as a Flycast request.

Official references: [Private networking](https://fly.io/docs/networking/private-networking/), [Flycast](https://fly.io/docs/networking/flycast/), and [private app autostart](https://fly.io/docs/blueprints/autostart-internal-apps/).

## Isolation and deployment

Set only required secrets on each app. Database attachment is also per app; follow the selected database reference rather than sharing a copied URL by default.

```bash
fly deploy --config fly.web.toml --app prod-example-web
fly deploy --config fly.api.toml --app prod-example-api
fly scale count 2 --app prod-example-web
```

The count is an example, not a recommendation. Deploy in dependency order, keep API compatibility across rolling versions, and verify both apps after each release.
