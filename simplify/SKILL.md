---
name: simplify
description: "Behavior-preserving cleanup of a working-tree or explicit base/head diff. Use after implementation or when asked to simplify current or committed changes."
---

# Simplify

Own behavior-preserving cleanup. Produce a smaller, clearer diff while keeping the implemented design intact.

## Scope

1. Resolve the initial diff: use the current working tree by default, or an explicit base/head comparison supplied by the user or calling skill for committed changes. For a committed comparison, record its merge base and head, and verify the head is checked out before editing. Inspect `git status --short`, the complete initial diff and stat, and relevant hand-authored untracked files.
2. Read the repository instructions that apply to each changed file, including root-to-file `AGENTS.md`, `CLAUDE.md`, and any documents they reference. Derive conventions and validation commands from those instructions, nearby code, package scripts, and CI configuration.
3. Treat the user's stated focus and intended changes as the boundary. Preserve unrelated work and include generated files only when the task owns their source and regeneration.

Scope is established when the initial diff is reproducible, every in-scope file and applicable instruction is named or inspected, and unrelated changes are identified. The final candidate diff is the post-edit working-tree diff for working-tree scope; for committed scope, it is the recorded merge base compared with the checked-out head plus its staged, unstaged, and relevant untracked cleanup edits.

## Preservation Contract

An edit qualifies only when evidence shows that all observable contracts remain equivalent:

- public APIs, types, inputs, outputs, request/response shapes, and serialization
- database schemas, persisted data meaning, migrations, and compatibility expectations
- errors, status codes, logging relied on by consumers, side-effect order, concurrency, and timing guarantees
- user-visible UI, interactions, accessibility semantics, and state transitions
- test meaning and coverage; test cleanup retains the same assertions while production equivalence remains independently evidenced

Apply the smallest local edit with affirmative equivalence evidence. Record uncertain, speculative, cross-cutting, or redesign-dependent ideas as skipped. This section is the authoritative preservation boundary for every lens and finding below.

## Reviewer Lenses

Review every changed hunk through all four lenses:

| Lens | Target | Evidence to inspect |
| --- | --- | --- |
| Conventions | Match established repository and domain patterns | Applicable instructions, neighboring implementations, imports, naming, typing, errors, tests |
| Simplicity | Remove accidental indirection and cognitive load | Redundant wrappers or state, nesting, stale comments, over-generalized functions, needless parameters |
| Duplication | Reuse a natural existing source of truth or a small local shared expression | Repeated parsing, validation, mapping, literals, branches, and nearby helpers without awkward dependencies |
| Dataflow | Make work and ownership direct while removing obvious waste | Repeated I/O, N+1 work, unnecessary sequencing or recomputation, leaked listeners, redundant updates |

Clear code outranks shorter code. Repository precedent outranks generic taste. Efficiency findings qualify when the simpler dataflow also satisfies the preservation contract.

## Finding Record

Maintain one coverage row per in-scope hunk: `path:line-range | conventions | simplicity | duplication | dataflow`. Mark each lens `clear` or list its finding IDs so a no-finding pass remains checkable.

Record each actionable opportunity before editing:

```yaml
id: S1
lens: conventions | simplicity | duplication | dataflow
location: path/to/file.ts:line
evidence:
  current: exact construct in the changed hunk
  precedent: nearby pattern, instruction, caller, or measured waste
issue: concrete accidental complexity
change: smallest proposed edit
preservation: contracts examined and why each remains equivalent
payoff: code, branch, dependency, state, or work removed
risk: low | medium | high
confidence: low | medium | high
disposition: applied | skipped
reason: evidence supporting the disposition
verification: command or inspection tied to this finding
```

A finding names local evidence and a concrete action. Apply findings only when preservation evidence is complete, confidence is high, and the edit is local and reviewable.

## Execution

1. Review every in-scope hunk against every lens, reading surrounding code and affected callers where equivalence depends on them.
2. Resolve overlapping findings into one smallest change. Re-read the current file immediately before editing so concurrent work remains intact.
3. Apply findings sequentially and update each disposition. Inspect the complete resulting diff after all edits.
4. Run `git diff --check`, repository-required checks, and the narrowest tests, type checks, lint, builds, or behavioral exercises that cover the edited surfaces. Expand validation when a failure or risk requires it.

## Completion Gate

Report `SIMPLIFIED` only when all statements are true:

- Every in-scope changed hunk has a recorded pass through all four lenses.
- Every finding has an `applied` or `skipped` disposition with evidence and a preservation rationale.
- Every applied edit satisfies every item in the preservation contract; no affected contract surface is unaccounted for.
- The final candidate diff includes every original in-scope hunk and every cleanup edit, contains only intended changes and simplifications, and leaves unrelated working-tree changes intact.
- `git diff --check`, every applicable repository-required check, and every selected targeted validation pass.
- Each omitted or unavailable check is named with the concrete reason and leaves no unsupported success claim.

If any statement is false, report `BLOCKED` and name each unmet statement. A valid no-edit result records exhaustive lens coverage and explains why no finding qualified.

## Report

Return the outcome, applied findings by file, skipped findings with reasons, validation commands and results, and totals. Keep explanations proportional to the evidence.
