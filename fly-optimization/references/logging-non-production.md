# Non-Production Logging

Use this profile only for environments without production data. If staging, preview, or development receives production data, use the production profile instead.

## Default posture

Write application logs to stdout/stderr and prefer live tailing for short-lived diagnosis:

```bash
fly logs --app staging-example
fly logs --app staging-example --json
```

Keep searchable retention and export proportional to the environment's lifetime and debugging need. Avoid copying production retention, alerting, or export infrastructure into disposable environments without a concrete requirement. Verify current Fly capabilities in [Logging overview](https://fly.io/docs/monitoring/logging-overview/) and [Search logs](https://fly.io/docs/monitoring/search-logs/). When export is selected, follow Fly's current [Export logs](https://fly.io/docs/monitoring/exporting-logs/) guidance and record the destination, credentials boundary, expected delivery delay, and recovery behavior.

## Diagnostic events

Emit the same versioned event names and core fields used in production so behavior can be reproduced across environments. Include `environment`, `service`, and correlation identifiers. Development-only detail may be more verbose, but the event contract remains compatible with production.

Use synthetic or scrubbed data. Exclude credentials, authorization headers, session tokens, and unnecessary personal data even when the environment is temporary. Broader developer access is acceptable only when the data classification permits it.

Validation is complete when a uniquely identified test event appears in every selected live-tail, search, or export destination; the environment field is correct; access matches the environment's data classification; retention matches its lifetime and debugging requirement; export interruption and recovery are tested when export applies; and sensitive sentinel values are absent from captured output.
