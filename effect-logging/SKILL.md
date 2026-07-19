---
name: effect-logging
description: Effect v4 logging and observability for APIs and backends. Use for structured logs and logger layers; typed error and Cause boundaries; request context and log spans; or distributed tracing integration.
---

# Effect v4 Logging and Observability

Use the project-pinned `effect` source first. If it does not answer an API question, check current upstream source; v4 is version-sensitive.

## Process

1. **Inspect.** Read the pinned Effect version, nearby conventions, error union, runtime layers, HTTP adapter, and telemetry sink. Continue only when each touched boundary and its data sensitivity are known.
2. **Route.** Load every matching reference before proposing or editing code:
   - Error classes, expected failures, external error normalization, `catchTag`, or `orDie`: read [`references/error-patterns.md`](references/error-patterns.md).
   - HTTP status/body/log policy, `Cause`, defects, composite causes, interruption, or framework adapters: read [`references/error-handler.md`](references/error-handler.md).
   - Built-in/custom loggers, privacy, log annotations, log spans, tracing, or test capture: read [`references/logger-setup.md`](references/logger-setup.md).
3. **Apply or review.** Keep business code on `Effect.log*`; map every domain error explicitly at the owning boundary; preserve interruption; and install logger/tracer implementations once at the runtime edge. Continue until every changed event, error variant, Cause reason, and metadata field has a deliberate policy.
4. **Verify.** Typecheck and test against the pinned package, then inspect emitted logs and responses for representative success, each domain error, a defect, a composite cause, and interruption. Review field names and sample values for privacy before completion.

## Defaults

- Model expected application failures with current `Schema.TaggedErrorClass<Self>()(...)` when schema-backed errors fit.
- Use `Effect.fn("Domain.operation")` for named operation boundaries. It creates a tracing span; it is not a log span.
- Use `Effect.annotateLogs` for inherited log fields and `Effect.withLogSpan` only for elapsed-time context in log output.
- Use `Effect.withSpan` and `Effect.annotateCurrentSpan` for distributed tracing. Configure a tracer exporter separately.
- Treat an error's typed status, HTTP status, log level, client body, and alert policy as separate decisions.
- Use static event names plus event-specific allowlisted fields. Raw bodies, headers, credentials, tokens, and arbitrary error objects are outside the allowlist.

## Severity

| Level | Use |
|---|---|
| `Trace` / `Debug` | Temporary or sampled diagnostic detail |
| `Info` | Normal lifecycle events and routine client outcomes |
| `Warn` | Degradation, throttling, retries, or conditions needing attention but not immediate investigation |
| `Error` | Failed dependencies, defects, or outcomes needing investigation |
| `Fatal` | Process-level loss of service where shutdown or restart is expected |

Do not infer severity from "typed" versus "defect" alone. A typed persistence failure can be `Error`; an ordinary not-found can be `Info` or unlogged.

## Completion Gate

Finish only when all of these are true:

- Every reachable domain error has an exhaustive status, safe body, log level, event name, and alert decision.
- Every Cause path handles all typed failures, defects, interruptions, and composite causes without collapsing an `Option` or swallowing cancellation.
- Every logger path deliberately handles event time, every Cause reason, fiber id, allowlisted annotations, and active log spans without leaking unrestricted payloads.
- Log annotations, log spans, and distributed tracing are named and configured as distinct mechanisms.
- Every changed API name and signature was checked against the project-pinned Effect source or current upstream v4 source, and every framework-shaped example is identified as framework-specific or pseudo-code.
