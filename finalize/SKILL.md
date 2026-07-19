---
name: finalize
description: "Finalize working-copy changes, an existing PR, or both into an objectively merge-ready state. Use for a final pass, hardening, or ship readiness."
context: fork
---

# Finalize

Own the last engineering pass on a working tree. The result is an objectively merge-ready tree or an explicit blocked outcome.

## Ownership and Authorization

Finalize may inspect repository and read-only PR state, edit intended working-tree files, and run safe local, test, sandbox, simulator, or preview validation. Its default stopping point is the merge-ready tree and report.

Version-control transitions and external side effects are opt-in. Perform a branch creation or switch, commit or amend, push, PR creation or update, review reply or resolution, merge, deployment, release, publication, or external communication only when the user explicitly authorizes that class of action in the current conversation. Invocation of `finalize` alone grants none of those actions.

The explicit PR-to-deployment lifecycle belongs to `code-review-loop`; hand off to it only when the user requests that lifecycle. Behavior-preserving cleanup belongs to `simplify`.

## Scope

1. Determine whether the evidence source is the working-copy diff, an existing PR diff, or both. Use the user's stated scope; ask only when ambiguity could mix unrelated work.
2. Inspect status, effective diff, changed files in full, relevant callers and tests, and applicable root-to-file repository instructions such as `AGENTS.md`, `CLAUDE.md`, and their references.
3. Identify unrelated working-tree changes and preserve them. Restrict edits to intended files and hunks unless an affected dependency requires a clearly evidenced fix.

Scope is established when every changed file belongs to the intended change or is explicitly excluded, and every applicable instruction has been inspected.

## Risk Tiers

Assign the highest tier whose condition applies and record the evidence:

| Tier | Conditions | Minimum depth |
| --- | --- | --- |
| Low | Local implementation or documentation change with no shared contract, persisted data, security, concurrency, infrastructure, or rollout effect | Inspect the changed surface and direct tests; run targeted repository checks |
| Medium | Multi-file or shared-path behavior, dependency/config changes, indirect consumers, meaningful failure modes, or a non-trivial rollback | Trace affected callers and states; run affected-package checks and exercise the changed behavior |
| High | Authentication/authorization, public API compatibility, schema or persisted data, migrations, transactions/concurrency, infrastructure/deployment, broad architecture, or hard rollback | Trace end-to-end and operational impact; validate integrations plus applicable migration, rollback, rollout, observability, and containment paths |

A small diff can be high risk. Missing evidence raises the tier rather than lowering it.

## Readiness Pass

### 1. Build the Impact Map

For every changed surface, record what changed, direct and indirect consumers, plausible failure modes, user or operational impact, and the validation that can detect each material regression. Classify every credible affected surface as checked, non-material with evidence, or blocked.

The impact map is complete when every changed file and every credible affected surface has one classification.

### 2. Resolve Shipping Findings

Investigate correctness, contracts, edge states, security, accessibility, failure handling, docs/configuration, rollout, and operability where the impact map makes them relevant. Record each finding with location, evidence, severity, proposed action, and verification.

Fix findings whose correct action is supported by repository evidence and fits the task boundary. Preserve product, API, migration, and rollout decisions as explicit blockers when intent cannot be inferred safely. Classify non-blocking follow-ups separately with the evidence that makes them safe to defer.

Every finding is resolved as fixed, non-blocking follow-up, or blocker; each disposition has a concrete reason.

### 3. Delegate Cleanup

Invoke `simplify` after correctness fixes, supplying the same working-tree or explicit base/head effective diff established for finalize. Its preservation contract, reviewer lenses, finding record, and completion gate are authoritative; finalize does not recreate or weaken them. A `BLOCKED` simplify result blocks finalize. A no-edit `SIMPLIFIED` result is valid when its exhaustive lens pass found no qualifying cleanup.

Cleanup is complete when `simplify` reports `SIMPLIFIED` and every cleanup edit appears in the final impact map and validation plan.

### 4. Handle PR-Only Evidence

When PR scope and read access are available, inspect required checks and every unresolved review thread. Convert correctness or shipping-risk feedback into findings, route behavior-preserving cleanup through `simplify`, and classify outdated, vague, stylistic, or out-of-scope feedback with a concise no-change rationale. Replying to or resolving a thread follows the authorization boundary.

PR evidence is complete when every unresolved thread and required check has a recorded disposition tied to the current PR head SHA.

### 5. Validate by Risk

Create a validation matrix before running checks: each material affected surface maps to a repository command or a safe behavioral exercise and an expected result. Use commands from repository instructions, package scripts, and CI rather than assuming a package manager.

Run the matrix plus `git diff --check`. For medium risk, include affected-package and real-flow coverage. For high risk, include the applicable integration and operational paths from the tier definition. Diagnose failures, fix those caused by the intended change, and rerun every invalidated row. Classify unrelated failures with evidence; a materially required unavailable or failing row is a blocker.

Validation is complete when every matrix row passes and every omitted check is shown to be non-material.

### 6. Audit the Final Tree

Re-read the complete diff and current status. Confirm that fixes address their findings, simplifications satisfy `simplify`, generated artifacts correspond to owned sources, documentation matches behavior, and unrelated user work remains intact.

The audit is complete when every final hunk maps to the intended change, a resolved finding, or an authoritative simplification finding.

## Completion Gate

Report `TREE_MERGE_READY` only when all statements are true:

- Scope is established and the risk tier has evidence.
- Every changed file and credible affected surface is classified in the impact map; every material surface is verified.
- Every finding and PR-only review item in scope has a recorded disposition, with zero unresolved blockers.
- `simplify` reports `SIMPLIFIED`, and every resulting edit is included in the impact map.
- `git diff --check`, every applicable repository-required check, and every validation-matrix row pass.
- Every final hunk is accounted for and all unrelated working-tree changes remain intact.
- The authorization boundary was observed, so the report distinguishes tree readiness from unpublished or unmerged remote state.

If any statement is false, report `BLOCKED` and enumerate each unmet statement, its evidence, and the next decision or action needed. Confidence labels never substitute for this gate.

## Report

Return the outcome, scope and risk evidence, fixed and deferred findings, surfaces verified, exact validation results, unverified blockers, and current tree/PR publication state. Keep follow-ups separate from blockers.
