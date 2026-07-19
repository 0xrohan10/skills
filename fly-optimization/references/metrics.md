# Metrics

Fly publishes built-in proxy, Machine, volume, and supported Postgres metrics. Custom Prometheus metrics require a reachable endpoint and `[metrics]` or per-group `[[metrics]]` configuration.

```toml
[[metrics]]
  processes = ["web"]
  port = 9091 # MEASURE: application metrics listener
  path = "/metrics"

[[metrics]]
  processes = ["worker"]
  port = 9092 # MEASURE: worker metrics listener
  path = "/metrics"
```

Bind custom metrics listeners to `0.0.0.0` as required by Fly's scraper. Keep the endpoint response below current platform limits and control label cardinality. Verify scrape interval, retention, cost, response limits, available series, and alerting support in current [Metrics documentation](https://fly.io/docs/monitoring/metrics/) rather than duplicating volatile values.

## Query API

Use a token scheme that matches the token type. `fly auth token` uses `Bearer`; tokens created by `fly tokens create` use `FlyV1`.

```bash
ORG_SLUG='<org-slug>'
TOKEN="$(fly auth token)"

curl --fail-with-body \
  --get "https://api.fly.io/prometheus/${ORG_SLUG}/api/v1/query" \
  --data-urlencode 'query=sum by (app, status) (increase(fly_edge_http_responses_count[5m]))' \
  --header "Authorization: Bearer ${TOKEN}"
```

The range makes this valid standard PromQL as well as MetricsQL. Check that the JSON response has `status: success` and non-empty results for an app that received traffic.

## Starting queries

Use measured objectives to set thresholds. These expressions are starting queries, not alert values:

```promql
sum by (app) (rate(fly_edge_http_responses_count{status=~"5.."}[5m]))
/
clamp_min(sum by (app) (rate(fly_edge_http_responses_count[5m])), 1e-9)

histogram_quantile(
  0.99,
  sum by (app, le) (rate(fly_app_http_response_time_seconds_bucket[5m]))
)

max by (app, instance) (fly_instance_exit_oom)
```

Also observe concurrency, CPU throttling, available memory, filesystem/volume use, restarts, queue depth, job age, database connections, and replication health as applicable. Fly's managed Grafana is for dashboards and exploration; current docs state that built-in alerting is not included, so select and test an external or self-hosted alert path when alerts are required.

Validation is complete when every configured target is scraped, each required query returns the expected labels, a synthetic failure crosses the chosen rule, and the notification reaches the accountable operator.
