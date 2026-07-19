# HTTP Error and Cause Boundaries

Read this reference when mapping domain errors to transport responses or inspecting full causes. The response types below are application-neutral pseudo transport shapes; adapt them to the project's HTTP framework instead of assuming Hono, Express, or Effect HTTP APIs.

## Exhaustive domain policy

Keep status, client body, severity, and event name in one exhaustive policy. This example intentionally treats routine client outcomes differently from throttling and dependency failures.

```ts
type DomainError =
  | { readonly _tag: "ValidationError"; readonly field: string }
  | { readonly _tag: "NotFoundError"; readonly resource: string }
  | { readonly _tag: "UnauthorizedError" }
  | { readonly _tag: "RateLimitedError"; readonly retryAfterSeconds: number }
  | { readonly _tag: "PaymentDeclinedError"; readonly reasonCode: string }
  | { readonly _tag: "PersistenceError"; readonly operation: string }
  | { readonly _tag: "PaymentProviderError"; readonly operation: string }

type ErrorPolicy = {
  readonly status: number
  readonly code: string
  readonly clientMessage: string
  readonly logLevel: "Info" | "Warn" | "Error"
  readonly event: string
  readonly alert: boolean
}

const absurd = (value: never): never => {
  throw new Error(`Unhandled domain error: ${String(value)}`)
}

const policyFor = (error: DomainError): ErrorPolicy => {
  switch (error._tag) {
    case "ValidationError":
      return { status: 400, code: "invalid_request", clientMessage: "Request validation failed", logLevel: "Info", event: "http.request.invalid", alert: false }
    case "NotFoundError":
      return { status: 404, code: "not_found", clientMessage: "Resource not found", logLevel: "Info", event: "http.resource.not_found", alert: false }
    case "UnauthorizedError":
      return { status: 401, code: "unauthorized", clientMessage: "Authentication required", logLevel: "Info", event: "http.request.unauthorized", alert: false }
    case "RateLimitedError":
      return { status: 429, code: "rate_limited", clientMessage: "Too many requests", logLevel: "Warn", event: "http.request.rate_limited", alert: false }
    case "PaymentDeclinedError":
      return { status: 422, code: "payment_declined", clientMessage: "Payment was declined", logLevel: "Info", event: "payment.declined", alert: false }
    case "PersistenceError":
      return { status: 503, code: "temporarily_unavailable", clientMessage: "Service temporarily unavailable", logLevel: "Error", event: "dependency.persistence.failed", alert: true }
    case "PaymentProviderError":
      return { status: 502, code: "payment_unavailable", clientMessage: "Payment service unavailable", logLevel: "Error", event: "dependency.payment.failed", alert: true }
    default:
      return absurd(error)
  }
}
```

Adding an error to `DomainError` now fails typechecking until its boundary policy is added. Keep response messages stable and safe; diagnose with allowlisted fields, not leaked error payloads.

## Cause-aware boundary

Effect v4 `Cause` can hold several `Fail`, `Die`, and `Interrupt` reasons. A single-error shortcut is valid only after proving the Cause has exactly one typed failure. `Cause.findErrorOption` returns an `Option<E>`; narrow it and use `.value`, never pass the `Option` itself to policy or response code.

```ts
import { Cause, Effect, Option } from "effect"

type HttpErrorResponse = {
  readonly status: number
  readonly body: {
    readonly error: {
      readonly code: string
      readonly message: string
    }
    readonly request_id: string
  }
}

const responseFromPolicy = (
  policy: ErrorPolicy,
  requestId: string,
): HttpErrorResponse => ({
  status: policy.status,
  body: {
    error: { code: policy.code, message: policy.clientMessage },
    request_id: requestId,
  },
})

const logDomainError = (policy: ErrorPolicy, error: DomainError) => {
  const log = policy.logLevel === "Info"
    ? Effect.logInfo
    : policy.logLevel === "Warn"
    ? Effect.logWarning
    : Effect.logError

  return log(policy.event).pipe(
    Effect.annotateLogs({
      error_tag: error._tag,
      http_status: policy.status,
      alert: policy.alert,
    }),
  )
}

const withHttpErrorBoundary = <A, R>(
  effect: Effect.Effect<A, DomainError, R>,
  requestId: string,
) =>
  effect.pipe(
    Effect.catchCause((cause) => {
      if (Cause.hasInterrupts(cause)) {
        return Effect.failCause(cause)
      }

      const error = Cause.findErrorOption(cause)
      const isSingleTypedFailure =
        cause.reasons.length === 1 &&
        Cause.isFailReason(cause.reasons[0])

      if (isSingleTypedFailure && Option.isSome(error)) {
        const domainError = error.value
        const policy = policyFor(domainError)
        return logDomainError(policy, domainError).pipe(
          Effect.as(responseFromPolicy(policy, requestId)),
        )
      }

      const response: HttpErrorResponse = {
        status: 500,
        body: {
          error: { code: "internal_error", message: "Internal server error" },
          request_id: requestId,
        },
      }

      return Effect.logError("http.request.failed").pipe(
        Effect.annotateLogs({
          reason_count: cause.reasons.length,
          cause_kind: Cause.hasDies(cause) ? "defect_or_composite" : "composite_failure",
        }),
        Effect.as(response),
      )
    }),
  )
```

This boundary makes four deliberate choices:

- Any interruption, including an interruption mixed with another reason, is re-failed unchanged so cancellation is not swallowed.
- Exactly one typed failure uses the exhaustive domain policy.
- Defects and composite non-interruption causes log a bounded classification and return a generic `500`; choosing one status from several failures would be lossy and order-dependent. Send a sanitized Cause to a separately access-controlled diagnostic sink only when the operational requirement justifies it.
- Client responses contain only stable public fields plus a request id. The full Cause remains available to Effect control flow; production logs receive only the explicitly sanitized diagnostic fields required by their sink policy.

If the application has a truthful policy for homogeneous composite failures, encode and test it explicitly rather than silently selecting the first failure.

## Framework and process edges

- Framework-specific adapters should run the Effect boundary before converting to the framework response type. Avoid duck-typing arbitrary thrown values after `runPromise` has already collapsed the Cause.
- Node-specific `unhandledRejection` and `uncaughtException` handlers live outside the Effect runtime. Use the process logger directly there, synchronously flush if the sink supports it, and terminate after an uncaught exception.
- Log and alert on rates and service-level impact, not merely on every `4xx`. Routine validation and not-found events usually belong in metrics or sampled `Info` logs.
