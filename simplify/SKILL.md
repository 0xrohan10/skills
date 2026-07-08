---
name: simplify
description: >
  Simplify current working-tree changes by applying behavior-preserving edits that follow codebase conventions, reduce duplication, improve clarity, and avoid broad rewrites.
---

# Simplify

Use this skill after an implementation pass to make the current working tree smaller, clearer, and more consistent with the existing codebase.

The goal is not to redesign the code. The goal is to remove accidental complexity.

## Inputs

- Current working-tree changes
- Any user-provided focus, such as:
  - reduce duplication
  - follow repo conventions
  - simplify route loaders
  - clean up API handlers
  - avoid public API changes

## Rules

Preserve behavior exactly.

Do not change:
- public APIs
- request/response shapes
- database schemas
- error semantics
- ordering of side effects
- user-visible UI behavior
- tests unless the existing implementation already changed behavior and tests must follow

Prefer:
- existing codebase conventions
- nearby patterns
- explicit boring code
- small edits
- local simplification

Avoid:
- broad rewrites
- clever compression
- new abstractions for future use
- extracting helpers used only once
- replacing clear code with dense code
- changing behavior because a cleaner design exists

## Workflow

### 1. Inspect the working tree

Review these:

    git status --short
    git diff --stat
    git diff --unified=80
    git ls-files --others --exclude-standard

Treat tracked diffs as the main scope.

For untracked files, inspect only hand-authored source, docs, and config files. Ignore build output, generated files, vendor files, lockfiles, and binaries unless directly relevant.

### 2. Launch focused reviewers

Use subagents where available. If the harness does not support subagents, run these reviews as separate passes.

Each reviewer only proposes changes. The parent or main agent decides what to apply.

#### Reviewer A: Codebase conventions

Look for changed code that works but does not fit the repo.

Check:
- neighboring files
- existing handlers, routes, services, components, and tests
- naming conventions
- import style
- error handling
- typing patterns
- file organization
- framework idioms already used in the repo

Good findings:
- new code bypasses an existing service or repository boundary
- route/API errors use a different shape than nearby code
- names do not match domain language
- imports or file layout differ from local practice
- tests are structured differently from adjacent tests

Skip generic style opinions.

#### Reviewer B: Duplication and reuse

Look for:
- duplicated logic introduced in the diff
- existing helpers, types, or constants that should be reused
- repeated validation, parsing, mapping, or formatting logic
- string literals that should use existing enums or constants
- copy-paste branches that can share a small local helper

Only suggest reuse when it genuinely reduces code and preserves behavior.

Do not import distant helpers if that creates an awkward dependency.

#### Reviewer C: Simplicity and readability

Look for:
- unnecessary variables
- redundant wrappers
- over-generalized functions
- nested conditionals that can become guard clauses
- confusing names
- stale or obvious comments
- unnecessary state, effects, or memoization
- parameters that are already available from context

Prefer clearer code, not merely shorter code.

#### Reviewer D: Efficiency and dataflow

Look for obvious waste introduced by the diff:

- repeated expensive work
- avoidable repeated DB, API, or file calls
- accidental N+1 patterns
- unnecessary sequential awaits
- missing cleanup for listeners, timers, or subscriptions
- state updates that fire even when nothing changed
- avoidable rerenders or recomputation

Skip micro-optimizations unless they also simplify the code.

### 3. Require structured findings

Each reviewer should return findings in this shape:

    {
      "findings": [
        {
          "category": "conventions | duplication | simplicity | efficiency",
          "file": "path/to/file.ts",
          "issue": "short description",
          "change": "specific proposed change",
          "risk": "low | medium | high",
          "confidence": "low | medium | high"
        }
      ]
    }

Reject vague findings.

A good finding is actionable and local.

### 4. Merge findings

Prioritize findings in this order:

1. Conventions
2. Simplicity
3. Duplication
4. Efficiency

Apply only findings that are:

- behavior-preserving
- high-confidence
- small enough to review
- consistent with nearby code
- unlikely to cause merge conflicts

Skip findings when:
- behavior could change
- the old code is ugly but intentional
- the proposed abstraction is speculative
- reviewers disagree and the safer choice is unclear
- the edit would require broad rewrites

### 5. Apply edits carefully

Apply edits sequentially, one file at a time.

Before each edit:
- read the current file
- confirm the change still applies
- confirm it is smaller or clearer than the existing version
- confirm it preserves behavior

Prefer surgical edits over whole-file rewrites.

After editing, inspect the final diff.

### 6. Verify

Run the narrowest useful checks available.

Prefer, in order:
- targeted tests for changed files
- package-level typecheck
- package-level lint
- existing cheap validation command

Do not run expensive integration suites unless explicitly requested.

If no obvious check exists, say so.

## Final response

Keep the final response short.

Use this format:

    Simplified:

    path/to/file.ts
      ✓ [Conventions] aligned error handling with nearby handlers
      ✓ [Simplicity] flattened nested branching
      ✓ [Duplication] reused existing parser helper

    Not applied:
      ⚠ [Efficiency] batching looked possible, but ordering semantics were unclear
      ⚠ [Duplication] similar helper exists, but it changes invalid-input behavior

    Verification:
      ✓ pnpm typecheck
      - No targeted tests found

    Totals:
      applied 3 · skipped 2

Do not include a long explanation unless the user asks.
