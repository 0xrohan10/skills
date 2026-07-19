# Production Logging

Use this profile for production and for any environment that contains production data. Determine retention, access, export, and redaction from incident-response, audit, legal, privacy, and recovery requirements before choosing Fly's built-in search or an owned sink.

## Capture and access

Write application logs to stdout/stderr. Fly captures the stream, and `fly logs` provides live tailing:

```bash
fly logs --app prod-example
fly logs --app prod-example --instance <machine-id>
fly logs --app prod-example --region <region>
fly logs --app prod-example --json
```

Grant production log access only to operators who need it. Record who can use live tail, searchable logs, and exported sinks; these may have different access boundaries.

## Retention and export

Fly's searchable-log backend, retention, beta status, and cost are volatile. Verify [Logging overview](https://fly.io/docs/monitoring/logging-overview/) and [Search logs](https://fly.io/docs/monitoring/search-logs/) during each design or review.

Compare the current service with the production retention and availability requirements. When it is insufficient, export to an approved sink using Fly's current [Export logs](https://fly.io/docs/monitoring/exporting-logs/) guidance. A log shipper is a separate private app consuming Fly's log stream; give it an appropriate restart policy, least-privilege credentials, and a durable sink. Test delivery interruption, duplicate handling, and recovery.

## Production events

Emit one JSON object per line with a versioned event name and stable fields such as `level`, `message`, `timestamp`, `environment`, `service`, `request_id`, and operation-specific identifiers. Add the process group when groups share an app stream.

Allowlist fields for each event. Exclude credentials, authorization headers, session tokens, raw request or response bodies, and personal data that is not required for the operational purpose. Apply redaction before export because downstream retention and access can exceed Fly's defaults.

Validation is complete when a uniquely identified production-safe test event appears in every required destination, access matches the approved operator list, verified retention satisfies the requirement, interruption recovery is tested for every exporter, and sensitive sentinel values are absent from captured output.
