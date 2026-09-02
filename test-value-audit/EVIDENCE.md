# Test Value Evidence

Use this reference to classify test value. Test length, age, simplicity, and passing status are not evidence of low value.

## Deletion gate

Classify a test as **DELETE** only when every condition is true:

1. It protects no distinct requirement, failure mode, boundary, or integration seam.
2. A surviving test provides equal or stronger protection, or the asserted behavior has no meaningful observable contract.
3. Removal leaves plausible regressions equally detectable and diagnosable.
4. It documents no non-obvious contract or prior production defect.
5. Its apparent coverage is incidental, duplicated, or disconnected from observable behavior.

Concrete replacement evidence names a surviving test and explains how its assertions detect the same regression. Shared executed lines and coverage percentages are not replacement evidence. Uncertain contracts require **KEEP** or human judgment.

## Low-value evidence

### Tautology

The test reproduces production logic, compares an imported value with itself, checks constructor inputs remain present, or verifies a mock's configured result. Valuable tests constrain the implementation independently.

### Weak assertion

The test only checks that execution succeeds, a value exists, a response exists, or an object is truthy when a stronger behavior matters. An important scenario with weak assertions is **REWRITE**; an unimportant scenario can be **DELETE**.

### Implementation detail

The test constrains private call order, internal shapes, incidental intermediate values, helper choice, or invocation counts with no observable consequence. Invocation is observable when it is itself the contract, such as charging once or suppressing an event after rollback.

### Mock theatre

The test configures collaborators and then confirms the same return values or calls. Mock-based tests retain value when they verify argument transformation, conditional or forbidden calls, consequential ordering, retries, idempotency, or error translation.

### Duplicate

Another test exercises the same input class, code path, observable result, failure mode, and integration boundary with equal or better diagnostics. Prefer the test with clearer setup, stronger assertions, and less mocking. Executing the same lines does not make two tests duplicates.

### Redundant permutation

Additional inputs exercise no new equivalence class. Distinct classes can include ordinary valid input, missing input, boundaries and their nearest values, malformed input, permissions, dependency failure, and duplicate operations. Repeated cases remain useful when exhaustive domain examples or a table-driven contract are intentional.

### Platform behavior

The test verifies language semantics, standard-library behavior, a stable dependency's own guarantee, or unconfigured framework behavior. Application configuration, wrappers, adapters, dependency assumptions, and upgrade regressions remain valuable contracts.

### Trivial pass-through

A getter, re-export, field mapping, or one-line delegation has no transformation, policy, compatibility contract, or credible independent failure. Externally visible, financial, security-sensitive, and easily miswired mappings retain value.

### Low-information snapshot

The snapshot is large generated noise, routinely accepted without review, duplicates focused assertions, or covers output with no stability contract. Focused snapshots can protect stable serialization, generated documents, protocol messages, and intentionally reviewed complex UI.

### Dead coverage

The test covers removed behavior, unreachable code, obsolete flags, or an abandoned implementation. A skipped test for still-required behavior is broken coverage, not a deletion candidate.

## Positive value evidence

A valuable unit test protects a business rule, transformation, boundary, state transition, authorization decision, error policy, idempotency rule, non-obvious orchestration, or regression-prone behavior.

A valuable integration test exercises a real seam:

- Application and database.
- HTTP route and application service.
- Serialization and deserialization.
- Transaction commit or rollback.
- Authentication or authorization enforcement.
- Queue or event production and consumption.
- Adapter and external-service contract.
- Database constraints, queries, or migrations.
- Components collaborating under realistic configuration.

Unit coverage of a business rule does not replace integration coverage of wiring, configuration, schemas, queries, serialization, or transactions. Consolidate integration tests only when they protect the same seam and scenario. A fully mocked "integration" test provides integration value only if a real seam remains under test.

## Protected contracts

Require strong replacement evidence for tests covering:

- A prior production defect.
- Authentication or authorization.
- Money, billing, accounting, or ledgers.
- Data loss or destructive operations.
- Transactions or concurrency.
- Idempotency or duplicate events.
- Time zones, date boundaries, or precision.
- Database constraints or migrations.
- Public API compatibility.
- Error paths or partial failures.
- Critical system connectivity.
