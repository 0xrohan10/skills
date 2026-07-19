# Process Groups

Use one Fly App when processes share an image, deploy, secrets, ownership, and app-level network exposure. Each process group runs on separate Machine(s), not beside the others in one Machine. `fly deploy` updates every group and creates at least one Machine for a newly defined group; scale groups independently after deployment.

Official reference: [Run multiple process groups](https://fly.io/docs/launch/processes/).

## Lifecycle model

Choose lifecycle by workload, not by whether the group has a health check:

- **Public or Flycast request service:** configure `[http_service]` or `[[services]]`. Fly Proxy can route requests and, when enabled, stop/suspend and wake Machines.
- **Continuous queue consumer, scheduler, or poller:** omit a service, use an `always` restart policy, and keep the required Machine count running. A top-level check observes health but does not control lifecycle.
- **Request-driven private worker:** place it in a private Flycast app when it must scale to zero. Producers must call `<app>.flycast`; direct `.internal` traffic bypasses Fly Proxy and cannot wake a stopped Machine.
- **Finite job:** use a one-off or scheduled Machine with an intentional exit and restart policy rather than pretending it is a service.

Fly Launch Machines default to `on-failure`; an always-on process without a service needs an explicit `always` restart policy. Verify current semantics in [Machine restart policy](https://fly.io/docs/machines/guides-examples/machine-restart-policy/) and [Fly Proxy autostop/autostart](https://fly.io/docs/launch/autostop-autostart/).

## Baseline configuration

Every number tagged `MEASURE` is a placeholder. Replace it from load tests, startup observations, failure objectives, and resource telemetry before applying.

```toml
app = "prod-example"
primary_region = "yyz"

[env]
  HOST = "0.0.0.0"
  NODE_ENV = "production"
  PORT = "3000" # MEASURE: application listen port
  WORKER_HEALTH_HOST = "0.0.0.0"
  WORKER_HEALTH_PORT = "4000" # MEASURE: worker health listener

[processes]
  web = "bun run start:web"
  worker = "bun run start:worker"

[http_service]
  processes = ["web"]
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

[[restart]]
  processes = ["worker"]
  policy = "always"

[checks]
  [checks.worker]
    processes = ["worker"]
    port = 4000 # MEASURE: worker health listener
    type = "http"
    method = "get"
    path = "/health"
    grace_period = "10s" # MEASURE: worst healthy startup time
    interval = "30s" # MEASURE: detection objective and check cost
    timeout = "5s" # MEASURE: healthy endpoint latency ceiling

[[vm]]
  processes = ["web"]
  cpu_kind = "shared"
  cpus = 1 # MEASURE: load test and CPU telemetry
  memory = "512mb" # MEASURE: peak RSS plus headroom

[[vm]]
  processes = ["worker"]
  cpu_kind = "shared"
  cpus = 1 # MEASURE: job throughput target
  memory = "1gb" # MEASURE: peak job RSS plus headroom
```

`auto_stop_machines = "off"` and `auto_start_machines = false` express an always-running web service. If measured demand supports autostop, enable stop or suspend and autostart together, then set `min_machines_running` from the latency and availability objective. `min_machines_running` only acts in the primary region and only when autostop is enabled.

## Process commands

Commands in `[processes]` supersede Docker `CMD`, not `ENTRYPOINT`. Verify each group starts the intended executable from the built image. A wrapper entrypoint must end in `exec "$@"` so Fly's process command and termination signals reach the application.

## Secrets and scaling

All groups receive all app secrets. Split apps when credentials differ.

```bash
fly scale count web=2 worker=1 --app prod-example
fly scale vm performance-2x --process-group worker --app prod-example
```

Counts and VM presets above are examples, not recommendations. Confirm actual state with `fly status --app prod-example` and inspect each Machine's restart policy with `fly machine status -d <machine-id>`.

## Checks

Service checks affect routing and deployments. Top-level checks monitor processes without services and require their listener on `0.0.0.0`. Neither substitutes for restart policy. Health endpoints should test the process's ability to do its work; include database connectivity only when loss of the database should make the process unhealthy.
