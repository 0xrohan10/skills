# Error Modeling and Recovery

Read this reference when defining errors, normalizing external failures, choosing failure versus defect, or adding typed recovery.

## Schema-backed typed errors

Current Effect v4 defines schema-backed yieldable errors with `Schema.TaggedErrorClass<Self>()(tag, fields)`. They carry `_tag`, extend `Error`, and can be yielded directly in `Effect.gen`.

```ts
import { Effect, Schema } from "effect"

class ValidationError extends Schema.TaggedErrorClass<ValidationError>()(
  "ValidationError",
  {
    field: Schema.String,
    issue: Schema.String,
  },
) {}

class NotFoundError extends Schema.TaggedErrorClass<NotFoundError>()(
  "NotFoundError",
  {
    resource: Schema.String,
  },
) {}

interface User {
  readonly id: string
}

declare const lookupUser: (id: string) => Effect.Effect<User | undefined>

const findUser = Effect.fn("User.find")(function* (id: string) {
  const user = yield* lookupUser(id)
  if (user === undefined) {
    return yield* new NotFoundError({ resource: "User" })
  }
  return user
})
```

Use the error union inferred by composition. Define a schema union only when a public wire, persistence, RPC, or schema-tooling boundary needs one.

```ts
class PaymentDeclinedError extends Schema.TaggedErrorClass<PaymentDeclinedError>()(
  "PaymentDeclinedError",
  { reasonCode: Schema.String },
) {}

class PaymentProviderError extends Schema.TaggedErrorClass<PaymentProviderError>()(
  "PaymentProviderError",
  {
    operation: Schema.String,
    cause: Schema.Defect(),
  },
) {}

const PaymentError = Schema.Union(
  PaymentDeclinedError,
  PaymentProviderError,
)
type PaymentError = typeof PaymentError.Type
```

## External failures

Normalize thrown or rejected values at the adapter boundary. Store only fields the application needs to route or diagnose the failure.

```ts
interface Charge {
  readonly id: string
}

declare const createProviderCharge: (
  orderId: string,
  amountCents: number,
) => Promise<Charge>

const chargeCard = (orderId: string, amountCents: number) =>
  Effect.tryPromise({
    try: () => createProviderCharge(orderId, amountCents),
    catch: (cause) =>
      new PaymentProviderError({
        operation: "payments.createCharge",
        cause,
      }),
  })
```

`Schema.Defect()` supplies an `unknown` runtime value with a JSON encoding when the schema encoder is explicitly used. It is lossy for values JSON cannot faithfully represent; it does not make direct `JSON.stringify(error)` safe or guarantee round-tripping. Prefer safe provider codes and operation labels over copying provider payloads into logs or client responses.

## Failure, defect, and severity

- **Typed failure:** an outcome a caller may handle, such as validation, not-found, denial, throttling, decline, dependency unavailability, or persistence failure.
- **Defect:** an invariant violation, bug, unexpected throw, or other failure outside the declared error channel.
- **Interruption:** cancellation, not an application error. Broad Cause handlers must preserve it.

Typed failures are not uniformly `400` or `Warn`. Transport and logging policy belongs to the owning boundary: validation may be `400` and `Info`, rate limiting `429` and `Warn`, while a typed database outage may be `503` and `Error`.

Use `Effect.orDie` only when the boundary deliberately promotes a typed failure to an unrecoverable defect, commonly during startup after no truthful recovery remains.

```ts
declare const loadRequiredStartupConfig: Effect.Effect<unknown, Error>

const start = loadRequiredStartupConfig.pipe(Effect.orDie)
```

## Typed recovery

Use typed recovery before Cause-level recovery. Catch only where the caller has a truthful fallback, alternate route, retry policy, or transport mapping.

```ts
const optionalUser = findUser("user-123").pipe(
  Effect.catchTag("NotFoundError", () => Effect.succeed(undefined)),
)

declare const processPayment: Effect.Effect<Charge, PaymentError>

const paymentResult = processPayment.pipe(
  Effect.catchTags({
    PaymentDeclinedError: (error) =>
      Effect.succeed({ accepted: false as const, code: error.reasonCode }),
    PaymentProviderError: () =>
      Effect.succeed({ accepted: false as const, code: "temporarily_unavailable" }),
  }),
)
```

`Effect.catch` handles all typed failures but leaves defects and interruption alone. Prefer `catchTag` or `catchTags` when variants need different behavior; let unhandled errors continue to the boundary policy in [`error-handler.md`](error-handler.md).
