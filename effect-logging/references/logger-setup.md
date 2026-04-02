# Logger Setup in Effect-TS

## Built-in Logger Options

Effect ships with several logger implementations you can swap in without any external dependencies.

### Logger.json (recommended for prod)

Outputs structured JSON to stdout. This is the simplest path to queryable logs.

```ts
import { Effect, Logger } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.logInfo("server started").pipe(
    Effect.annotateLogs({ port: 3000 })
  )
})

// Provide the JSON logger at the edge
program.pipe(
  Effect.provide(Logger.json),
  Effect.runPromise
)
```

Output:
```json
{"level":"INFO","message":"server started","timestamp":"2025-01-15T...","annotations":{"port":3000},"spans":[]}
```

### Logger.pretty (recommended for dev)

Human-readable colored output for terminal development.

```ts
program.pipe(
  Effect.provide(Logger.pretty),
  Effect.runPromise
)
```

### Logger.structured

Similar to JSON but returns structured objects (useful if you want to post-process before shipping).

## Custom Logger with Pino

If you need pino-specific features (child loggers, custom serializers, redaction), build a custom logger:

```ts
import { Logger, Effect, Layer } from "effect"
import pino from "pino"

const pinoInstance = pino({
  level: "info",
  // Redact sensitive fields before they hit the transport
  redact: {
    paths: ["*.password", "*.token", "*.ssn", "*.healthCardNumber"],
    censor: "[REDACTED]",
  },
  serializers: {
    err: pino.stdSerializers.err,
  },
})

const PinoLogger = Logger.make(({ logLevel, message, annotations, spans, date }) => {
  const level = logLevel.label.toLowerCase()
  const annotationObj = Object.fromEntries(annotations)
  const spanEntries = spans.map(
    (s) => `${s.label}=${Date.now() - s.startTime}ms`
  )

  const logFn = pinoInstance[level] ?? pinoInstance.info
  logFn.call(pinoInstance, {
    ...annotationObj,
    spans: spanEntries.length > 0 ? spanEntries : undefined,
    msg: typeof message === "string" ? message : JSON.stringify(message),
  })
})

// Create a Layer that replaces the default logger
export const PinoLoggerLive = Logger.replace(Logger.defaultLogger, PinoLogger)
```

### Providing the Logger

Provide once at the application entry point — never scatter logger setup through your codebase:

```ts
import { Effect, Layer } from "effect"

const AppLive = Layer.mergeAll(
  PinoLoggerLive,
  DatabaseLive,
  HttpClientLive,
  // ... other layers
)

const main = program.pipe(Effect.provide(AppLive))

Effect.runPromise(main)
```

## Swapping Loggers per Environment

Use config or environment variables to select the logger layer:

```ts
const loggerLayer =
  process.env.NODE_ENV === "production"
    ? PinoLoggerLive      // structured JSON for prod
    : Logger.pretty        // colored terminal for dev

const AppLive = Layer.mergeAll(loggerLayer, DatabaseLive, HttpClientLive)
```

## Test Logger (Capturing Logs in Memory)

For tests, capture log output in memory so you can assert on what was logged:

```ts
import { Effect, Logger, Ref, Array as Arr, Layer } from "effect"

const makeTestLogger = Effect.gen(function* () {
  const logs = yield* Ref.make<Array<{ level: string; message: string; annotations: Record<string, unknown> }>>([])

  const logger = Logger.make(({ logLevel, message, annotations }) => {
    const entry = {
      level: logLevel.label,
      message: typeof message === "string" ? message : JSON.stringify(message),
      annotations: Object.fromEntries(annotations),
    }
    // Ref.update is synchronous and safe to runSync here — don't use this pattern for effectful operations
    Effect.runSync(Ref.update(logs, Arr.append(entry)))
  })

  return { logger, logs } as const
})
```

Usage in tests:

```ts
import { it, expect } from "@effect/vitest"

it.effect("logs payment processing", () =>
  Effect.gen(function* () {
    const { logger, logs } = yield* makeTestLogger
    const testLoggerLayer = Logger.replace(Logger.defaultLogger, logger)

    yield* processPayment("order-123").pipe(Effect.provide(testLoggerLayer))

    const captured = yield* Ref.get(logs)
    expect(captured).toContainEqual(
      expect.objectContaining({
        level: "INFO",
        message: expect.stringContaining("payment completed"),
      })
    )
  })
)
```

## Log Level Filtering

Set the minimum log level to reduce noise in prod:

```ts
import { Logger, LogLevel } from "effect"

// Only show info and above in prod
const filteredLogger = Logger.minimumLogLevel(LogLevel.Info)

// Combine with your logger implementation
const ProdLoggerLive = Layer.mergeAll(PinoLoggerLive, filteredLogger)
```

## PII Redaction Strategy

For PHIPA/PIPEDA compliance, handle PII at the serialization boundary — not in business logic:

1. **Pino redaction paths** (shown above) — declare sensitive field paths and pino redacts them before they hit the transport.
2. **Custom annotation sanitizer** — if using the built-in JSON logger, wrap it to strip known sensitive keys before output.
3. **Don't log request/response bodies by default** — they're large and likely contain PII. Log summaries (status code, content-length, duration) instead.

The guiding principle: business logic should log freely using annotations. The logger layer is responsible for ensuring nothing sensitive leaves the process.
