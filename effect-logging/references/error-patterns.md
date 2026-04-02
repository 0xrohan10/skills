# Error Patterns in Effect-TS

## Schema.TaggedErrorClass

Define all domain errors with `Schema.TaggedErrorClass`. These are serializable, type-safe, yieldable directly in generators (no `Effect.fail` wrapper needed), and carry a `_tag` for pattern matching.

```ts
import { Schema } from "effect"

class ValidationError extends Schema.TaggedErrorClass("ValidationError")(
  "ValidationError",
  {
    field: Schema.String,
    message: Schema.String,
  }
) {}

class NotFoundError extends Schema.TaggedErrorClass("NotFoundError")(
  "NotFoundError",
  {
    resource: Schema.String,
    id: Schema.String,
  }
) {}

class PaymentDeclinedError extends Schema.TaggedErrorClass("PaymentDeclinedError")(
  "PaymentDeclinedError",
  {
    orderId: Schema.String,
    reason: Schema.String,
    gateway: Schema.String,
  }
) {}
```

Tagged errors are yieldable — you can yield them directly in a generator to fail:

```ts
const findUser = Effect.fn("findUser")(function* (id: string) {
  const user = yield* db.query(`SELECT * FROM users WHERE id = $1`, [id])
  if (!user) {
    yield* new NotFoundError({ resource: "User", id })
  }
  return user
})
```

## Schema.Defect for External Errors

When calling third-party SDKs (stripe, twilio, external APIs), the errors they throw are unknown — random JS objects, strings, Error subclasses with non-serializable properties. `Schema.Defect` wraps any unknown thrown value into a serializable form.

```ts
class StripeError extends Schema.TaggedErrorClass("StripeError")(
  "StripeError",
  {
    endpoint: Schema.String,
    statusCode: Schema.Number,
    idempotencyKey: Schema.optional(Schema.String),
    error: Schema.Defect,
  }
) {}

const chargeCard = (orderId: string, amount: number) =>
  Effect.tryPromise({
    try: () =>
      stripe.charges.create({ amount, currency: "cad", source: orderId }),
    catch: (error) =>
      new StripeError({
        endpoint: "/v1/charges",
        statusCode: 500,
        idempotencyKey: orderId,
        error,
      }),
  })
```

`Schema.Defect` handles:
- JS Error instances → serialized as `{ name, message }`
- Unknown values → string representation
- Circular references → safely handled

This matters for logging because you can safely `JSON.stringify` the entire error without worrying about serialization blowups.

## Error Unions

Group related errors into unions for handler signatures:

```ts
const PaymentError = Schema.Union(PaymentDeclinedError, StripeError)
type PaymentError = typeof PaymentError.Type
```

## Expected Errors vs Defects

**Expected errors (the E channel):** Domain failures the caller can handle. Validation errors, not-found, permission denied, rate limits, payment declined. These are typed, recoverable, and should be logged at `warn` level.

**Defects:** Bugs, invariant violations, things that should never happen. A null where your types say there shouldn't be one, a config that fails to load at startup. These terminate the fiber and are handled once at the system boundary. Log at `error` or `fatal`.

```ts
// Expected: caller can handle this
const findUser = Effect.fn("findUser")(function* (id: string) {
  const row = yield* db.query(id)
  if (!row) yield* new NotFoundError({ resource: "User", id })
  return row
})

// Defect: if config can't load, nothing works — die immediately
const main = Effect.gen(function* () {
  const config = yield* loadConfig.pipe(Effect.orDie)
  yield* Effect.logInfo(`starting on port ${config.port}`)
})
```

## Recovery with catchTag / catchTags

Handle specific error types by their tag:

```ts
const recovered = program.pipe(
  Effect.catchTag("NotFoundError", (err) =>
    Effect.gen(function* () {
      yield* Effect.logWarning(`${err.resource} not found: ${err.id}`)
      return fallbackValue
    })
  )
)
```

Handle multiple types at once:

```ts
const recovered = program.pipe(
  Effect.catchTags({
    NotFoundError: (err) => Effect.succeed(null),
    ValidationError: (err) =>
      Effect.gen(function* () {
        yield* Effect.logWarning("validation failed").pipe(
          Effect.annotateLogs({ field: err.field })
        )
        return null
      }),
  })
)
```

## Catch-All with Effect.catch

For catching everything in the error channel:

```ts
const recovered = program.pipe(
  Effect.catch((error) =>
    Effect.gen(function* () {
      yield* Effect.logError("request failed", error)
      return fallbackValue
    })
  )
)
```
