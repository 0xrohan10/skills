---
name: ssot-audit
description: >-
  Audit a software codebase for competing sources of truth, duplicated business
  rules, manually synchronized contracts, split-brain state, multiple writers,
  and configuration or schema drift. Use for single-source-of-truth (SSOT),
  architecture-consistency, and cross-layer authority audits; not for ordinary
  duplicate-code cleanup.
---

# Single source of truth audit

## Goal

For each significant domain fact, invariant, rule, contract, configuration value,
or mutable state, determine whether one authority owns the decision. Check that
other representations derive from it or have explicit limits on their authority.

SSOT is not DRY. Identical code is not necessarily an SSOT violation. Different
implementations can still be competing authorities. A single authority does not
require a single physical representation or a single place where a rule is
*enforced*.

## Default behavior and constraints

- **Read-only by default.** Inspect and report; do not edit application code,
  schemas, migrations, or configuration unless separately asked to remediate.
- Do not assume the database, backend, frontend, or shared package is always the
  correct authority. Choose an owner based on domain boundaries and who has
  the right to decide or change the fact.
- Prefer one-way derivation through imports, code generation, API responses,
  or event and projection pipelines over manual synchronization of separate copies.
- Exclude generated files, dependency trees, snapshots, build artifacts, and
  vendored code from initial discovery, but trace generated consumers back
  to their origin when relevant.
- Read repository-specific instructions, architecture documentation, and
  relevant tests. Record the audited revision or branch and any inaccessible
  areas. Do not claim complete coverage unless you inspected all relevant areas.
- Never flag something only because a variable name, literal, or code fragment
  appears twice. A finding needs semantic equivalence and a plausible route to
  inconsistency, or evidence of an existing inconsistency.

## Workflow

### 1. Establish scope and map the architecture

1. Identify repository layout, languages, packages, entry points, dependency
   direction, schemas, tooling, APIs, state stores, queues, jobs, and configuration
   sources. Read `AGENTS.md` and local conventions when present.
2. Identify the important bounded contexts and workflows where errors have
   serious consequences. Examples include authorization, pricing, billing, and
   lifecycle or status changes.
3. Note what is in scope, excluded, generated, external, and not inspected.
4. Begin with a broad inventory, then inspect high-risk domains in detail. Do not
   use the context window to catalog repeated literals with little audit value.

### 2. Inventory candidate truths by concept

Look for:

- Domain rules include calculations, eligibility, permissions, policies,
  validations, defaults, thresholds, and state transitions.
- Contracts include database schemas, ORM models, API schemas, DTOs, generated
  types, enums, statuses, error codes, and shared identifiers.
- Data ownership includes records, computed fields, materialized views, caches,
  event-driven projections, external services, and mutable state stores.
- Configuration includes environment defaults, runtime flags, reference tables,
  constant maps, tenant settings, and deployment values.
- Cross-layer behavior includes backend, frontend, and worker implementations
  of the same decision or transformation.

Group candidates by business meaning, not just spelling. Two things called
`status` may have different meanings. A `priceForLane()` function and a UI
`estimatedTotal` expression may implement the same rule.

### 3. Trace an authority graph for each candidate

For each concept, answer:

1. **What is the truth?** Define the fact or invariant in plain language.
2. **Who owns it?** Find the sources allowed to define or change it.
3. **Who writes it?** Trace independent write paths, including jobs, hooks,
   migrations, event consumers, and admin interfaces.
4. **Who derives or enforces it?** Trace calculations, validators, generated
   artifacts, UI previews, caches, and projections.
5. **Who consumes it?** Trace endpoints, components, jobs, reports, and tests.
6. **What keeps replicas aligned?** Check for shared imports, generated contracts,
   synchronization mechanisms, runtime queries, or invariant tests. Note when
   alignment depends only on developers remembering to update each copy.

Use code search with `rg` or `git grep`. Use symbol references and call hierarchies
where available, and read relevant files. Follow actual control and data flow.
Search both definitions and consumers; looking only at the definitions is not
sufficient.

### 4. Find competing authorities

Prioritize these patterns:

- Multiple independently editable implementations of the same business rule.
- Different writers able to change the same fact without an explicit ownership,
  ordering, reconciliation, or conflict-resolution policy.
- Manually mirrored schemas, enums, defaults, lookup tables, and API contracts.
- Frontend state and server or database state both treated as definitive.
- Copy-pasted conditional branches, conversions, or calculations that can drift.
- Configuration values duplicated across runtime environments, code defaults,
  deployment files, database settings, or background workers.
- Cached or derived values that writers can change independently of their sources.
- A hidden fallback or alternate code path that redefines the authoritative rule.

For every candidate, check whether the apparent duplicate is a deliberate
representation, an independent decision, or an unproven similarity.

### 5. Eliminate false positives

Do not automatically report these as violations:

- Server authorization or validation checks repeated at another trust boundary.
  Multiple enforcement points can be necessary. Compare the underlying policy.
- Client-side validation for UX when the server remains authoritative.
- Code generated from a canonical schema or interface.
- Cache entries, read models, materialized views, and event projections with
  explicit rules for derivation, invalidation, and reconciliation.
- Historical or legally meaningful snapshots, such as the price when an invoice
  was issued. These should not change when today's pricing rules change.
- Same-looking concepts that legitimately differ by bounded context, tenant,
  time, version, or lifecycle stage.
- Tests and fixtures that independently assert a contract. Their independence
  can help detect accidental changes.

If duplication is valid only under an undocumented assumption, label it
"needs verification", not a confirmed defect.

### 6. Validate, rank, and propose a fix

A confirmed finding must contain:

- A one-sentence statement of the violated or threatened invariant.
- At least two relevant file and line references that show what independently
  determines the same truth. A documented path with multiple writers also qualifies.
- A concrete drift mechanism or demonstrated difference, plus affected paths.
- The proposed canonical owner and a derivation strategy that respects dependency
  direction.
- Minimal remediation steps, effects on migrations, backfills, and versioning, and
  regression tests that would prevent recurrence.

Classify findings as:

- Confirmed divergence. Definitions or outputs currently conflict.
- Competing authorities. Separate sources currently define the same rule or fact,
  but can change independently.
- Fragile synchronization. Replicas depend on manual updates or lack sufficient
  checks that they agree.
- Needs verification. Evidence suggests a problem, but intent or call paths
  remain unclear.

Rank severity as critical, high, medium, or low. Rank confidence separately as
high, medium, or low. Severity should reflect the consequence of drift, not the
number of duplicate lines. Prioritize security, money, persistence integrity,
permissions, externally visible contracts, and frequent-change rules.

## Output format

Use `references/report-template.md`. Return:

1. A short assessment of which major concepts have clear ownership and which do
   not. State revision, scope, coverage limitations, and whether this was static
   analysis only.
2. A ranked finding index with severity, confidence, category, and exact code
   locations.
3. One evidence-backed explanation per finding that states the invariant,
   authorities, relevant write, derivation, and read paths, drift scenario,
   owner recommendation, smallest safe fix, and tests.
4. An "Intentional replication / not-a-violation" section with major examples.
   Explain why those defensive checks and snapshots are not violations.
5. A remediation order that respects dependency direction and avoids creating
   a large cross-domain `shared` module.
6. Unverified candidates and unaudited areas. Never turn absence of evidence
   into a claim of absence.

If the repository is large, return the highest-impact findings and state what
you covered. Do not claim to have assessed every file. Prefer evidence of a few
real violations over a long list of string matches.

## Modes

- Audit is the default mode. Inspect and report only.
- In a focused audit, limit discovery to the named domain or workflow. Cross
  package boundaries when needed to trace the truth.
- For a remediation plan, specify ordered changes, contracts, and tests. Do not
  modify code.
- For implementation, proceed only when explicitly requested. Apply the smallest
  approved architecture change and verify dependent consumers and tests.
