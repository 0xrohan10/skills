---
name: typescript-structure
description: TypeScript structure through narrow domain modules and explicit dependency graphs. Use when designing or refactoring module boundaries, public APIs, domain types, imports, or Effect service architecture.
---

# TypeScript Structure

Apply these rules to every module created or changed. Project conventions take precedence where they already make a clear choice.

## Modules Follow Types

- Give every important domain type a dedicated module.
- Name the exported type after the domain concept.
- Keep constructors, operations, comparison, encoding, and decoding with that type.
- Export the smallest interface used by real clients. Keep representation and helpers private.
- Draft the exports and one client call site before implementing the module.

## Keep Interfaces Uniform

- Use the same names for the same operations across modules: `make`, `decode`, `encode`, `equals`, `compare`.
- Put the module's primary value first: `update(user, patch)`.
- Put callbacks and options last: `map(users, transform)`.
- Make the default operation safe. Use `OrThrow` only for an explicitly throwing variant: `find` and `findOrThrow`.
- Introduce a capability interface when generic code consumes it.

## Encode Valid States

- Replace boolean flags and state-dependent optional fields with discriminated unions.
- Exhaustively switch on every union. Close the default branch with `never` or the project's equivalent.
- Brand primitives whose domain invariants matter and expose validated construction.
- Put expected failure in `Effect`, `Result`, or another explicit return type.
- Decode `unknown` where untrusted data enters the program.

## Keep Names Explicit

- Use named imports or stable module qualifiers so identifier provenance is visible.
- Import internal dependencies from their owning leaf module.
- Restrict barrels to curated package entry points; use leaf imports for the internal dependency graph.
- Keep imports acyclic. Extract a lower-level module when a cycle reveals a shared primitive.
- Use domain-specific `equals`, `compare`, and hashing rather than representation-based shortcuts.

## Make Intent Checkable

- Put API documentation on exported declarations; put implementation rationale beside the relevant code.
- Make deliberately discarded synchronous results explicit with `void`.
- Observe every promise through `await`, return, or an explicit runtime boundary.
- Enforce formatting, import, exhaustiveness, and ignored-result conventions with project tooling.

## Effect Modules

When a changed module uses Effect, read [the Effect structure rules](references/effect-style.md). Verify APIs against the installed Effect version.

## Completion Gate

Before finishing, verify every changed module:

- gives each important type one owner;
- exposes only client-required operations;
- matches analogous modules in names, argument order, and failure behavior;
- makes invalid states and expected failures visible in types;
- uses explicit, acyclic imports;
- passes the project's formatter, typecheck, lint, and tests.
