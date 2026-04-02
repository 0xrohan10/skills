---
name: effect-logging
description: Logging, error handling, and observability patterns for Effect-TS API applications. Use this skill when the user is building an API or backend service with Effect-TS and needs guidance on structured logging, error modeling, tracing, log annotations, or observability setup. Also trigger when the user asks about Effect error handling patterns (Schema.TaggedErrorClass, defects vs failures, Cause), request-scoped logging context, Effect.fn tracing, log levels in Effect, or swapping logger implementations via layers. Trigger even if the user just mentions "logging" or "observability" alongside Effect/effect-ts — don't wait for them to name specific APIs.
---

# Effect-TS Logging & Observability

This skill provides idiomatic patterns for logging, error handling, and observability in Effect-TS API applications. It follows the conventions from [effect.solutions](https://effect.solutions) and prioritizes structured, fiber-aware, composable logging.

## Core Principles

1. **Errors are typed and tracked** — Effect's `Effect<A, E, R>` signature means the error channel is part of the type. Use this to distinguish expected failures from unexpected defects structurally, not with runtime flags.
2. **Logging is a first-class effect** — `Effect.logInfo(...)` and friends are effects that compose, respect fiber context, and carry annotations automatically. Don't reach for an external logger directly in business logic.
3. **Context flows down the fiber tree** — Use `Effect.annotateLogs` and `Effect.withLogSpan` to attach request-scoped metadata. Deeply nested services inherit annotations without explicit threading.
4. **Logger implementation is swappable** — Business logic calls `Effect.logInfo(...)`. The actual destination (stdout, JSON, pino, axiom) is determined by the Layer provided at the edge.

## When to Reach for This Skill

- Setting up structured logging in a new Effect API
- Modeling domain errors with Schema.TaggedErrorClass
- Adding request-scoped tracing/correlation to an existing Effect app
- Building a centralized error handler for an HTTP layer (e.g., hono, express)
- Choosing between `Effect.logWarning` vs `Effect.logError` for different failure modes
- Wrapping third-party SDK errors for safe serialization
- Swapping logger implementations between dev/test/prod

## Error Modeling

Use `Schema.TaggedErrorClass` (not `Data.TaggedError`) for all domain errors. Schema-based errors are serializable, type-safe, and integrate with `Schema.Defect` for wrapping unknown failures.

See `references/error-patterns.md` for:
- Schema.TaggedErrorClass definitions
- Schema.Defect for wrapping external SDK errors
- Expected errors vs defects — when to use each
- Error unions and catchTag/catchTags recovery
- The `Effect.orDie` pattern for unrecoverable failures

## Logging Patterns

### Use Effect.fn for Automatic Tracing

`Effect.fn` creates named, traced functions. Every invocation gets a span with call-site information — no manual `withLogSpan` needed at function boundaries.

```ts
const processPayment = Effect.fn("processPayment")(
  function* (orderId: string, amount: number) {
    yield* Effect.logInfo("charging payment")
    const charge = yield* stripeCharge(orderId, amount)
    yield* Effect.logInfo("payment completed").pipe(
      Effect.annotateLogs({ chargeId: charge.id })
    )
    return charge
  }
)
```

`Effect.fn` also accepts a second argument for cross-cutting transforms (retry, timeout) that stay co-located with the function definition.

### Request-Scoped Annotations

Attach metadata at the HTTP handler level. Every log line emitted within the effect — including from deeply nested services — inherits these annotations automatically.

```ts
const handleRequest = (req: Request) =>
  processOrder(req.body).pipe(
    Effect.annotateLogs({
      requestId: req.id,
      userId: req.userId,
      method: req.method,
      path: req.path
    }),
    Effect.withLogSpan("handleRequest")
  )
```

### Nested Spans for Timing

Use `withLogSpan` within a function to trace sub-operations. Spans nest and produce timing output like `handleRequest=245ms processOrder=220ms charge=180ms`.

```ts
const processOrder = (order: Order) =>
  Effect.gen(function* () {
    yield* validateOrder(order).pipe(Effect.withLogSpan("validate"))
    yield* chargePayment(order).pipe(Effect.withLogSpan("charge"))
    yield* sendConfirmation(order).pipe(Effect.withLogSpan("notify"))
  }).pipe(Effect.withLogSpan("processOrder"))
```

### Log Levels

| Level   | When to use                                                                 |
|---------|-----------------------------------------------------------------------------|
| `debug` | Local dev or on-demand investigation. Chatty by design.                     |
| `info`  | Normal operational events. Request lifecycle, job enqueued, webhook received.|
| `warn`  | Expected/operational failures. 4xx responses, retries, rate limits.         |
| `error` | Unexpected failures impacting users. Defects, infrastructure failures.      |
| `fatal` | Process-level failures. Unhandled rejections, uncaught exceptions.          |

Default prod level: `info`. If you're at `debug` in prod by default, signal-to-noise is garbage.

### What to Log vs What Not to Log

**Log:** request method/path/status/duration, auth events (login, failure, token refresh), business-critical state transitions, outbound service calls with timing, queue/job lifecycle.

**Don't log:** request/response bodies by default (PII risk, size), auth tokens/credentials, full SQL with parameters, health check endpoints (noise). In PHIPA/PIPEDA contexts, redact sensitive fields via a custom serializer or don't log them at all.

## Centralized Error Handling

See `references/error-handler.md` for the complete HTTP error handler pattern, covering:
- Matching on typed failures vs defects using Cause
- Logging operational errors at `warn`, defects at `error`
- Never leaking internal details to the client (generic 500 + request_id)
- Process-level safety nets (unhandledRejection, uncaughtException)

## Logger Implementation

See `references/logger-setup.md` for:
- Using Effect's built-in `Logger.json` for structured output
- Building a custom pino-backed logger via `Logger.make`
- Swapping implementations via `Logger.replace` and Layer composition
- Test logger that captures logs in memory for assertions

## Operational Concerns

- **Don't log in hot loops** — Log batch summaries, not per-item. This applies even though `Effect.logInfo` is lazy; it still creates work for the fiber runtime.
- **Consistent field names** — Pick a convention (snake_case for log fields is common) and enforce it across all services. A shared logger wrapper or annotation helper helps.
- **Sampling high-frequency errors** — If your database goes down, you'll generate thousands of identical error logs per minute. Implement rate-limiting or circuit-breaker logic around error logging for known failure modes.
- **Log shipping** — Don't rely on ephemeral container filesystems. Ship structured JSON to a queryable backend (axiom, datadog, grafana loki). On fly.io, stdout goes to the built-in log drain.
