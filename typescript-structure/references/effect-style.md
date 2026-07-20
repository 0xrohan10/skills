# Effect Style

Effect v4 APIs may shift between beta versions, so inspect the project's installed source before adapting the examples.

## Service Module

Give a domain capability one canonical ES module containing its contract, tag, errors, named operations, and default layer. Keep schemas, codecs, SDK details, and construction helpers private when clients do not need them.

```ts
import { Context, Effect, Layer, Schema } from "effect"

export class NotFound extends Schema.TaggedErrorClass<NotFound>()(
  "NotFound",
  { id: UserId },
) {}

export interface Interface {
  readonly get: (id: UserId) => Effect.Effect<User, NotFound>
}

export class Service extends Context.Service<Service, Interface>()(
  "@app/UserRepo",
) {}

export const layer = Layer.effect(
  Service,
  Effect.gen(function* () {
    const sql = yield* SqlClient.SqlClient

    const get = Effect.fn("UserRepo.get")(function* (id: UserId) {
      // Decode rows and classify infrastructure failures here.
    })

    return Service.of({ get })
  }),
)

export * as UserRepo from "./user-repo.js"
```

The final self-export creates the optional canonical `UserRepo.Service`, `UserRepo.layer`, and `UserRepo.NotFound` surface. Use it when the project's compiler, linter, and runtime support the pattern. Otherwise, retain the same module identity through `import * as UserRepo` at call sites rather than introducing a TypeScript `namespace`.

## Contract Before Interpreter

Sketch leaf service contracts first. Write higher-level orchestration against those contracts, then add live and test interpreters. Public service methods normally have `R = never`: the layer acquires lower-level requirements once and closes over them.

This keeps dependency information at two useful boundaries:

- the service contract states what callers may do and how it may fail;
- the layer states what the implementation requires and how it is acquired.

Map SDK, HTTP, SQL, filesystem, and configuration failures into domain-specific tagged errors before they leave the service boundary.

## Named Operations

Wrap public service methods and non-trivial workflows with `Effect.fn("Domain.operation")`. The name should identify the domain boundary in traces and logs. Apply whole-operation policies such as spans, error classification, retries, timeouts, and cleanup at that boundary.

## Data Ownership

- Default domain records to `Schema.Struct` with a same-name TypeScript interface.
- Use branded schemas for constrained identifiers and scalar values.
- Use `Data.TaggedEnum` for internal control states.
- Use `Schema.TaggedUnion` for states that cross persistence or transport boundaries.
- Use `Schema.TaggedErrorClass` for failures carried by Effect.
- Decode unknown input at adapters; construct trusted values through the schema's supported constructor.

Confirm exact constructors and type projections against the installed Effect version.

## Layer Graph

Choose constructors by acquisition semantics:

- `Layer.succeed` for an implementation that already exists;
- `Layer.sync` for lazy synchronous construction;
- `Layer.effect` for effectful acquisition;
- `Layer.effectContext` when one acquisition intentionally exposes multiple services;
- `Layer.unwrap` when runtime configuration selects a layer.

Build named, flat, topologically ordered layer subgraphs. Construct a parameterized resource layer once and reuse that value so Layer memoization shares the resource by identity.

Use composition operations deliberately:

- `Layer.provide` satisfies and hides an implementation dependency;
- `Layer.provideMerge` satisfies it and keeps it available downstream;
- `Layer.mergeAll` combines independent layers for downstream consumers.

Provide the assembled application layer at the runtime entry point. Business workflows depend on service contracts rather than provisioning their own implementations.

## Scoped Ownership

The layer that acquires a resource owns its release. A layer that owns a listener, stream, or worker forks it into that layer's scope so acquisition can complete and shutdown remains structured. Expose lifecycle controls only when starting and stopping are themselves domain operations.

## Test Interpreters

Substitute explicit test layers for module mocks. Reusable fakes implement the production contract and may expose a separate test-only control service for inspection or synchronization. Use Effect synchronization primitives and test clocks in place of sleeps. Acquire fresh layers per test unless sharing an expensive resource is deliberate.

## Effect Gate

Before completing structural work, verify every changed Effect module:

- has one canonical service identity and an intentional export surface;
- keeps implementation dependencies in its layer rather than leaking them through service methods;
- names observable operations with domain-qualified `Effect.fn` labels;
- translates boundary data and failures before they enter domain orchestration;
- participates in one explicit, centrally provided layer graph;
- owns acquired resources and long-lived fibers through scope;
- can be substituted by an explicit test layer;
- uses APIs verified against the installed Effect version.
