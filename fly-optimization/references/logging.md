# Logging on Fly.io

Fly handles log collection at the platform level. You don't need a sidecar, log shipper, or helper process — just write to stdout and Fly takes care of the rest.

---

## How It Works

```
your app (stdout)
  → fly's init process (pid 1 in the VM)
    → Vector (on the host)
      → NATS cluster
        → Quickwit (indexed, 30 day retention)
          → Grafana (search UI)
```

Every Machine in every app in your org has its stdout captured and routed through this pipeline automatically. stderr is captured too.

---

## Accessing Logs

### Live tail (CLI)

```bash
# tail all machines for an app
fly logs --app prod-myapp

# filter to a specific machine
fly logs --app prod-myapp --instance <machine-id>

# filter to a region
fly logs --app prod-myapp --region yyz

# json output
fly logs --app prod-myapp --json
```

### Search (Grafana)

The Fly dashboard includes a Grafana-backed log search UI with 30 days of retention. Accessible from the app's monitoring section in the dashboard.

Useful for:
- Searching across time ranges
- Correlating logs across machines
- Finding specific errors after the fact

---

## Structured Logging

Emit JSON to stdout. Grafana can parse and filter on structured fields, making search significantly more useful than raw text.

```typescript
// instead of:
console.log(`request completed in ${ms}ms`)

// do:
console.log(JSON.stringify({
  level: "info",
  msg: "request completed",
  duration_ms: ms,
  path: req.path,
  status: res.status,
}))
```

Fly doesn't require a specific schema — any valid JSON line gets indexed as structured. Pick a consistent shape across your apps and stick to it.

Recommended baseline schema:

```typescript
type LogLine = {
  level: "debug" | "info" | "warn" | "error" | "fatal"
  msg: string
  ts: string           // ISO 8601 (Fly adds its own, but yours is useful for precision)
  service: string      // app name or process group name
  request_id?: string
  duration_ms?: number
  [key: string]: unknown  // additional context fields per operation
}
```

Use this as your starting point and extend with domain-specific fields as needed. The important thing is that `level`, `msg`, `ts`, and `service` are consistent across all apps in the org — this makes cross-service log correlation work.

---

## Process Groups and Logging

With process groups, each group's logs appear in the same `fly logs` stream for the app. Fly adds metadata (machine ID, region) to each log line so you can distinguish which process group a log came from — but it helps to also include a `service` or `process` field in your structured logs:

```typescript
console.log(JSON.stringify({
  level: "info",
  service: "worker",  // or "web"
  msg: "job completed",
  job_id: jobId,
}))
```

With separate apps, logs are naturally separated by app name — `fly logs --app prod-myapp-web` vs `fly logs --app prod-myapp-api`.

---

## What Not To Do

- **Don't write logs to files** — Fly won't capture them, and volumes aren't the right place for logs
- **Don't run a separate logging process** — Fly handles collection; a sidecar adds complexity for no gain
- **Don't suppress stdout** — some frameworks default to buffered output or file logging in production; override this to ensure logs flow
- **Don't log sensitive data** — secrets, tokens, PII; logs are retained 30 days and accessible to anyone with org access

---

## Debug Logging for Deployments

```bash
LOG_LEVEL=debug fly deploy
```

Prints verbose platform logs during deploy — useful when a deploy is hanging or a health check isn't passing.
