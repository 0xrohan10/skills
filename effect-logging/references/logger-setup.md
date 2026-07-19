# Logger, Context, and Tracing Setup

Read this reference when selecting logger implementations, adapting another logging library, setting privacy rules, capturing test logs, or distinguishing logging from tracing. The APIs shown are current Effect v4 APIs.

## Three distinct mechanisms

| Mechanism | APIs | Purpose |
|---|---|---|
| Log annotations | `Effect.annotateLogs`, `References.CurrentLogAnnotations` | Inherited structured fields on log events |
| Log spans | `Effect.withLogSpan`, `References.CurrentLogSpans` | Active labels and elapsed milliseconds in log output |
| Distributed tracing | `Effect.fn`, `Effect.withSpan`, `Effect.annotateCurrentSpan`, tracer layers | Parent/child trace spans, propagation, export, and cross-service correlation |

`Effect.fn("Domain.operation")` creates a tracing span. `Effect.withLogSpan` only adds elapsed-time context to logs. Neither installs an OpenTelemetry exporter. `Logger.tracerLogger` records log events on the current trace span; keep or omit it deliberately when replacing the logger set.

## Built-in v4 loggers

`Logger.consoleJson` and `Logger.consolePretty()` write every supplied message, Cause, and annotation to the console. `Logger.formatJson` and `Logger.formatStructured` format values but do not write them by themselves. Treat the built-ins as development or trusted-input choices; they are not a production privacy boundary.

```ts
import { Effect, Logger, References } from "effect"

const DevelopmentLoggerLive = Logger.layer([
  Logger.consolePretty(),
  Logger.tracerLogger,
])

const runnable = program.pipe(
  Effect.provide(DevelopmentLoggerLive),
  Effect.provideService(References.MinimumLogLevel, "Debug"),
)
```

`Logger.layer([...])` replaces the active logger set by default. Pass `{ mergeWithExisting: true }` only when duplicate output is intended and understood. For production, use an adapter with an explicit event-field allowlist such as the Pino example below, then test the complete sink path with sensitive sentinels.

`Logger.tracerLogger` forwards all current log annotations to trace span events. Include it only when every annotation producer obeys the same field allowlist; otherwise omit it until the trace-event path has equivalent filtering.

## Request context

Use static event names and attach only allowlisted fields. Annotations flow through the provided effect and child fibers without manually threading a logger.

```ts
type RequestLogContext = {
  readonly request_id: string
  readonly method: string
  readonly route: string
}

const withRequestLogging = <A, E, R>(
  effect: Effect.Effect<A, E, R>,
  context: RequestLogContext,
) =>
  effect.pipe(
    Effect.annotateLogs(context),
    Effect.withLogSpan("http.request"),
    Effect.withSpan("http.request", { kind: "server" }),
  )
```

This `RequestLogContext` is an application-neutral shape, not a framework request type. Derive `route` from the matched route template, not a raw URL containing identifiers or query parameters.

## Pino adapter with complete runtime context

In v4, `Logger.Options` contains `date`, `cause`, `fiber`, `logLevel`, and `message`. Log annotations and log spans are fiber references, so a custom adapter reads them with `fiber.getRef(References.CurrentLogAnnotations)` and `fiber.getRef(References.CurrentLogSpans)`.

```ts
import { Cause, Logger, References } from "effect"
import pino from "pino"

const pinoLogger = pino({ level: "info" })

const allowedAnnotationKeys: ReadonlySet<string> = new Set([
  "alert",
  "cause_kind",
  "error_tag",
  "http_status",
  "method",
  "operation",
  "request_id",
  "reason_count",
  "route",
  "service",
])

const allowlistedAnnotations = (
  annotations: Readonly<Record<string, unknown>>,
): Record<string, unknown> =>
  Object.fromEntries(
    Object.entries(annotations).filter(([key]) => allowedAnnotationKeys.has(key)),
  )

const summarizeCause = (cause: Cause.Cause<unknown>) =>
  cause.reasons.map((reason) => {
    switch (reason._tag) {
      case "Fail": {
        const error = reason.error
        const errorTag =
          typeof error === "object" &&
          error !== null &&
          "_tag" in error &&
          typeof error._tag === "string"
            ? error._tag
            : "untagged"
        return { kind: "failure", error_tag: errorTag }
      }
      case "Die":
        return { kind: "defect" }
      case "Interrupt":
        return {
          kind: "interruption",
          interruptor_fiber_id: reason.fiberId,
        }
    }
  })

const writePino = (
  level: string,
  fields: Record<string, unknown>,
  message: string,
) => {
  switch (level) {
    case "Trace":
      return pinoLogger.trace(fields, message)
    case "Debug":
      return pinoLogger.debug(fields, message)
    case "Warn":
      return pinoLogger.warn(fields, message)
    case "Error":
      return pinoLogger.error(fields, message)
    case "Fatal":
      return pinoLogger.fatal(fields, message)
    case "Info":
    default:
      return pinoLogger.info(fields, message)
  }
}

const PinoLogger = Logger.make(
  ({ cause, date, fiber, logLevel, message }) => {
    const annotations = allowlistedAnnotations(
      fiber.getRef(References.CurrentLogAnnotations),
    )
    const logSpansMs = Object.fromEntries(
      fiber
        .getRef(References.CurrentLogSpans)
        .map(([label, startedAt]) => [label, date.getTime() - startedAt]),
    )
    const messages = Array.isArray(message) ? message : [message]
    const event = messages.length === 1 && typeof messages[0] === "string"
      ? messages[0]
      : "effect.log.invalid_message"

    return writePino(
      logLevel,
      {
        timestamp: date.toISOString(),
        effect_fiber_id: fiber.id,
        ...annotations,
        log_spans_ms: logSpansMs,
        cause: cause.reasons.length === 0 ? undefined : summarizeCause(cause),
      },
      event,
    )
  },
)

export const PinoLoggerLive = Logger.layer([PinoLogger])
```

The adapter deliberately:

- preserves the Effect event time instead of substituting adapter wall-clock time;
- records the current fiber id and computes active log-span durations against that event time;
- represents every reason in a composite Cause, while omitting typed-error payloads;
- records bounded defect classifications and keeps interruptions distinct without serializing thrown values, messages, or stacks;
- drops annotation keys outside the explicit allowlist and omits `Logger.tracerLogger` so it cannot bypass that filter.

The adapter accepts one static string as the event name and replaces any other message shape instead of serializing arbitrary payloads. If an incident workflow requires defect detail, sanitize it through a separately defined policy and send it to a dedicated access-controlled sink; this adapter remains bounded. Normalize third-party throws before they enter the error channel, restrict sink access/retention, and add deployment-specific redaction for known secrets. Redaction is defense in depth; it is not a substitute for the event-field allowlist.

## Test capture

`Logger.make` is synchronous, so tests can snapshot the values needed for assertions and install the logger with `Logger.layer`.

```ts
import { Effect, Logger, References } from "effect"

type CapturedLog = {
  readonly level: string
  readonly messages: ReadonlyArray<unknown>
  readonly annotations: Readonly<Record<string, unknown>>
}

const captured: Array<CapturedLog> = []
const TestLogger = Logger.make(({ fiber, logLevel, message }) => {
  captured.push({
    level: logLevel,
    messages: Array.isArray(message) ? [...message] : [message],
    annotations: {
      ...fiber.getRef(References.CurrentLogAnnotations),
    },
  })
})

const TestLoggerLive = Logger.layer([TestLogger])

const testProgram = program.pipe(
  Effect.provide(TestLoggerLive),
)
```

Assert event name, severity, and allowlisted fields. Add explicit tests for Cause, fiber id, date, and log-span handling when those are part of a custom adapter's contract.
