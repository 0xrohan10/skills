# Centralized Error Handling in Effect HTTP APIs

## The HTTP Error Handler Pattern

In any Effect-based API (hono, express, or a raw HTTP server), you want a single error-handling layer that catches anything that escapes your route handlers. This is where the expected-vs-defect distinction pays off.

```ts
import { Cause, Effect, Option } from "effect"

// Top-level error handler middleware
const withErrorHandler = <A, E, R>(effect: Effect.Effect<A, E, R>) =>
  effect.pipe(
    Effect.catchAllCause((cause) => {
      if (Cause.isFailureType(cause)) {
        // Typed failure from the E channel — operational/expected error
        const error = Cause.failureOption(cause)
        return Effect.gen(function* () {
          yield* Effect.logWarning("request failed with expected error").pipe(
            Effect.annotateLogs({
              error: JSON.stringify(Option.getOrElse(error, () => "unknown")),
              causeType: "failure",
            })
          )
          return toErrorResponse(error, 400)
        })
      }

      // Defect — something genuinely broke
      return Effect.gen(function* () {
        yield* Effect.logError("unexpected defect").pipe(
          Effect.annotateLogs({
            defect: Cause.pretty(cause),
            causeType: "defect",
          })
        )
        // Never leak internal details to the client
        return genericErrorResponse(500)
      })
    })
  )
```

### Key Decisions

**Operational errors → `logWarning`, specific status code, safe error message to client.**
These are 4xx-class: validation failures, not-found, auth failures, rate limits. The system is working as designed. The client gets a useful error message (but not internal details like stack traces).

**Defects → `logError`, generic 500, no internal details.**
The client gets `{ "error": "internal server error", "requestId": "abc-123" }` and nothing else. Your logs get the full `Cause.pretty(cause)` which includes the defect, the fiber trace, and any annotations.

### Hono Integration Example

```ts
import { Hono } from "hono"
import { Effect } from "effect"

const app = new Hono()

app.onError((err, c) => {
  const requestId = c.get("requestId")

  // At this boundary, typed Effect errors have been unwrapped into plain JS objects.
  // Duck-typing is the pragmatic choice here — Effect's type system doesn't reach into
  // framework error handlers. Match on _tag + statusCode to identify your domain errors.
  if ("_tag" in err && "statusCode" in err) {
    logger.warn({ err, requestId }, err.message)
    return c.json({ error: err.message, requestId }, err.statusCode)
  }

  // Unknown — treat as defect
  logger.error({ err, requestId }, "unhandled exception")
  return c.json({ error: "internal server error", requestId }, 500)
})
```

### Cause is Rich

`Cause` captures more than just the error — it records the full failure trace including:

- **Sequential composition** — if multiple effects failed in sequence
- **Parallel composition** — if concurrent fibers both failed
- **Interruption** — if a fiber was interrupted
- **The defect itself** — with stack trace

`Cause.pretty(cause)` gives a human-readable rendering of the entire causal chain, which is invaluable when you have fibers forking and joining.

## Outbound Call Failures

When calling external services, log more than just "call failed":

```ts
const callExternalApi = Effect.fn("callExternalApi")(
  function* (endpoint: string, payload: unknown) {
    const start = Date.now()
    const result = yield* Effect.tryPromise({
      try: () => fetch(endpoint, { method: "POST", body: JSON.stringify(payload) }),
      catch: (error) =>
        new ExternalApiError({
          service: "payments",
          endpoint,
          duration_ms: Date.now() - start,
          error,
        }),
    })
    yield* Effect.logInfo("external call completed").pipe(
      Effect.annotateLogs({
        service: "payments",
        endpoint,
        status: result.status,
        duration_ms: Date.now() - start,
      })
    )
    return result
  }
)
```

Log the duration, status code, service name, and endpoint. Do NOT log full request/response bodies (PII, size) or auth headers.

## Process-Level Safety Nets

These should never fire in a healthy app. If they do, it's a bug.

```ts
import { Effect } from "effect"

// In your main entry point
process.on("unhandledRejection", (reason) => {
  // Use your logger directly here since we're outside the Effect runtime
  logger.fatal({ err: reason }, "unhandled promise rejection")
})

process.on("uncaughtException", (err) => {
  logger.fatal({ err }, "uncaught exception — shutting down")
  process.exit(1) // State is unreliable, must exit
})
```

`fatal` level because the process is in an unknown state. If these fire, treat it as a P0 bug to investigate.

## Alert Strategy

The expected/defect split enables a sane alerting strategy:

- **Don't alert on expected errors.** A spike in 4xx responses might warrant a dashboard metric, but individual validation failures or not-founds are not incidents.
- **Alert on defects.** These are real bugs or infrastructure failures. Page someone.
- **Alert on error rate thresholds.** If your 4xx rate suddenly spikes 10x, that's worth investigating even though individual 4xxs aren't alertable — it might indicate a broken client deployment or a regression.
- **Alert on `fatal` always.** Process-level failures are always incidents.

Without the expected/defect distinction, you end up with alert fatigue — your error channel becomes a firehose of 400 bad requests mixed in with actual database outages, and eventually everyone just ignores it.
