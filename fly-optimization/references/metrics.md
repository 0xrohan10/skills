# Metrics on Fly.io

Fly includes a fully managed metrics stack — no extra infrastructure, no sidecar processes. Everything lands in Grafana at [fly-metrics.net](https://fly-metrics.net), the same place as your logs.

---

## How It Works

Two tiers:

**Built-in metrics** — automatically collected from every machine, zero config. Covers proxy-level and VM-level data.

**Custom metrics** — you expose a Prometheus `/metrics` endpoint from your app, add a `[metrics]` block to fly.toml, and Fly scrapes it.

Both are stored in managed VictoriaMetrics (Prometheus-compatible) and queryable via Grafana or the API.

---

## Built-in Metrics

Automatically published from every app. Labelled with `app`, `region`, `host`, `instance`.

### Proxy — `fly_edge_` and `fly_app_`

Edge metrics (before your app):
```
fly_edge_http_responses_count{status}
fly_edge_http_response_time_seconds{status}   # histogram
fly_edge_tcp_connects_count
fly_edge_tcp_disconnects_count
fly_edge_tls_handshake_errors{servername}
```

App metrics (after the proxy reaches your machine):
```
fly_app_concurrency
fly_app_http_responses_count{status}
fly_app_http_response_time_seconds{status}    # histogram
fly_app_connect_time_seconds                  # histogram
fly_app_tcp_connects_count
```

### VM — `fly_instance_`

Derived from `/proc` — CPU, memory, disk, network at the machine level.

`fly_instance_up = 1` confirms the VM is reporting correctly.

### Exit — `fly_instance_exit_`

Crucial for debugging and right-sizing:

```
fly_instance_exit_code    # 0 = normal, non-zero = error
fly_instance_exit_oom     # 1 = killed by OOM killer
```

`fly_instance_exit_oom` is the first thing to check when machines are mysteriously restarting. If it's firing, your VM is undersized.

---

## Custom Metrics

Expose a standard Prometheus `/metrics` endpoint from your app, then configure fly.toml to scrape it.

### fly.toml — single process

```toml
[metrics]
  port = 9091
  path = "/metrics"
```

### fly.toml — per process group

```toml
[[metrics]]
  port = 9091
  path = "/metrics"
  processes = ["web"]

[[metrics]]
  port = 9092
  path = "/metrics"
  processes = ["worker"]
```

Each process group can expose metrics on a different port. Fly scrapes each independently.

The metrics port does not need to be in `[[services]]` — it's scraped internally, not through the proxy.

---

## Dashboards

**fly-metrics.net** — managed Grafana preconfigured with your Prometheus datasource and built-in dashboards. Log in with your Fly account, switch orgs in the bottom-left.

Use the Explore panel for ad-hoc queries during debugging. Build custom dashboards for anything you expose via custom metrics.

---

## Querying the API Directly

```
https://api.fly.io/prometheus/<org-slug>/api/v1/query
```

```bash
TOKEN=$(fly auth token)
ORG=your-org-slug

curl "https://api.fly.io/prometheus/$ORG/api/v1/query" \
  --data-urlencode 'query=sum(increase(fly_edge_http_responses_count)) by (app, status)' \
  -H "Authorization: Bearer $TOKEN"
```

Useful for scripting alerts or pulling metrics into external tooling.

---

## What to Actually Monitor

**Note:** Fly uses VictoriaMetrics (MetricsQL), a PromQL superset. `rate()` and `increase()` just work — no `irate()` workarounds or Grafana's `$__rate_interval` magic needed. Write standard PromQL queries and they work.

The built-ins cover most of what you need without any app code:

- **Error rate**: `fly_edge_http_responses_count{status=~"5.."}` — alert if this spikes
- **Latency**: `fly_app_http_response_time_seconds` — p99 across your app
- **Concurrency**: `fly_app_concurrency` — useful for tuning `soft_limit`/`hard_limit` in fly.toml
- **OOM kills**: `fly_instance_exit_oom` — alert on any value > 0 in prod
- **Instance restarts**: `fly_instance_exit_code` — non-zero values worth investigating

### Starting-point alert thresholds

Tune these for your app, but these are reasonable defaults:

```
# 5xx rate exceeds 5% of total requests
rate(fly_edge_http_responses_count{status=~"5.."}[5m])
  / rate(fly_edge_http_responses_count[5m]) > 0.05

# Any OOM in prod is alertable
fly_instance_exit_oom > 0

# p99 latency above 2 seconds
histogram_quantile(0.99, rate(fly_app_http_response_time_seconds_bucket[5m])) > 2
```

For custom metrics, instrument anything business-critical that the proxy layer can't see — queue depth, job processing rate, cache hit rate, etc.
