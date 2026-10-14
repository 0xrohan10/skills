---
name: finalize
description: "Prepare the smallest viable PR. Assess its blast radius, preserve regression protection, and run simplify, unslop, test pruning, and adversarial code review."
context: fork
---

# Finalize

Produce the smallest viable PR that meets the intended requirements. Assess what the change can affect and verify that tests detect regressions. Remove unnecessary complexity without removing required behavior or useful tests. Prefer clarity and safety over fewer lines.

## Permissions

Inspect repository and read-only PR state, edit intended files, and run safe checks. Preserve unrelated user work. Branch changes, commits, pushes, PR or ticket updates, review replies, merges, deployments, and other external actions require explicit authorization in the current conversation. Invoking finalize does not authorize those actions.

## 1. Establish scope and blast radius

- Determine whether the scope is the working copy, a PR, or both. Read status, the full diff, changed files, repository instructions, relevant callers, and tests. For a PR, record the merge base and head SHA. Include local edits in the candidate diff.
- If the project has Linear MCP configured, require a linked ticket. Find it in the user's request, branch, commits, or PR. Read its description, acceptance criteria, and relevant comments through MCP. Ask if the ticket is missing or ambiguous. Block if you cannot read it. Define intended behavior and scope from the ticket, not from what the code happens to do.
- For each changed part, identify direct and indirect consumers, shared contracts, stored data, and operational effects. List possible failures and the checks that detect them. Mark each affected part as checked, not at risk with supporting evidence, or blocked.
- Assign the highest applicable risk tier. A small diff can be high risk. Missing evidence raises risk.

| Risk | Conditions | Required validation |
| --- | --- | --- |
| Low | Local change without shared-contract or operational effects | Targeted checks and direct tests |
| Medium | Shared paths, indirect consumers, dependency or configuration changes, or difficult rollback | Affected-package checks and tests of actual workflows |
| High | Authentication, authorization, public APIs, persisted data, migrations, concurrency, infrastructure, or hard rollback | End-to-end integrations and applicable migration, rollback, rollout, observability, and containment checks |

Use optional tools when they help assess the blast radius:

- Use `calldiff` for call-path changes. Run `calldiff --help` for supported syntax. Compare the recorded base with the current candidate using `calldiff diff`. Use `reach` or `tree` to trace affected entrypoints and callers. Record refs, paths, entrypoints, and depth limits. Treat output as supporting evidence, not proof that every dependency or runtime path was found. Inspect omitted paths separately.
- Use `dev-review-map` for structural or high-risk changes when before-and-after dependency maps would help. Supply exact base and head commits, validate both maps, and save them only in Git notes. Push notes only with explicit authorization. Maps cover committed changes, so inspect local edits separately.

Neither tool replaces ticket checks, tests, or adversarial review. If an optional tool is unavailable or blocked, report the limitation and assess the blast radius directly.

Fix correctness and release issues within scope when repository evidence supports the fix. Record where each issue occurs, how you confirmed it, its severity, the action taken, and how you checked the result. Block when a product or contract decision needs clarification.

For bug fixes, apply `principle-fix-root-causes`. Reproduce the failure and confirm that the fix removes its cause rather than hiding the symptom.

## 2. Run the preparation sequence

Load these skills and run them in order on the current diff, including earlier edits. Keep the same scope and supply any linked ticket's requirements. Follow each skill's evidence and completion rules. Report `BLOCKED` if a required skill is missing or a pass is blocked. A pass needs no edits if its review finds none to make.

1. Run `simplify` for behavior-preserving cleanup. Apply `principle-minimize-reader-load` to remove unnecessary layers and reduce hidden or mutable state. Require `SIMPLIFIED`.
2. Run `unslop` on text in the diff. Remove AI writing patterns and preserve meaning. Block if its instructions conflict with finalize's permissions or behavior-preservation requirements.
3. Run `test-value-audit` in cleanup mode on changed tests and tests affected by changed behavior. Apply `principle-test-behavior-not-implementation`. Call the interface users call and assert the result or effect they observe. Rewrite weak assertions for important behavior. Read neighboring tests to check overlap. Delete tests only when they meet the audit's deletion rules, and name surviving tests that detect the same regressions. Keep production code unchanged. Preserve uncertain tests and unrelated suites. Size, mocking, test count, and coverage percentage alone do not determine value.
4. Run `adversarial-code-review` on the full candidate without editing it. Require the rules the change must preserve, concrete failure cases, evidence for attacks the code prevents, and an `Approve` verdict. After the review, fix supported findings. Do not dismiss a correctness failure as style.

Update the blast-radius map and validation plan after each edit. Repeat affected passes in order. Reuse evidence only if it still applies. Adversarial approval must cover the final candidate.

Within these passes, apply `principle-type-system-discipline` when types change and `principle-make-operations-idempotent` when retries, crashes, or duplicate requests matter. When workers share mutable data, apply `principle-separate-before-serializing-shared-state`. Check whether they can own separate data before adding locks. Do not add another sequence of passes.

## 3. Check PR evidence and validate

For a PR, inspect required checks and every unresolved review thread at the current head SHA. Record an action or no-change reason for each item, with evidence. Recheck if the remote head changes. Replies and thread resolution require authorization.

For each material risk, choose a repository command or safe behavior check and state the expected result. Use commands from repository instructions, package scripts, and CI. Run these checks, all required repository checks, and `git diff --check` at the assigned risk tier.

Apply `principle-prove-it-works`. Run the affected workflow or inspect the actual output before claiming success. Compilation, cached results, and agent reports alone are not proof. For large changes, apply `principle-build-the-lever` with a small repeatable verification script and keep its output available to the reviewer.

For a linked ticket, show which code meets each requirement and acceptance criterion, and which check verifies it. Confirm that the final candidate fixes the reported problem without unexplained changes outside the ticket. Block on missing, conflicting, or unclear requirements.

Fix failures caused by the candidate and rerun affected checks. Record unrelated failures with evidence. Block if a required material check fails or cannot run, or if a material contract lacks regression protection.

## Completion and report

Read the final diff and status again. Report `TREE_MERGE_READY` only when:

- Every final hunk belongs to the intended change or a recorded fix. Unrelated work is intact, generated files match their sources, and documentation matches behavior.
- The blast-radius map accounts for every affected part. All material risks have passed validation, and all required checks pass.
- All four passes meet their completion rules for the final candidate. Every deleted test has surviving regression coverage.
- Every finding and PR item in scope has a supported resolution or safe follow-up. No blocker remains.
- If Linear MCP is configured, you have read the linked ticket and verified every requirement and acceptance criterion against the final candidate.
- No action exceeded the user's authorization. State tree readiness separately from publication or merge status.

Otherwise report `BLOCKED`. Give evidence and the next action for each unmet condition.

Report the outcome, scope, risk, and results of each pass. Include test-pruning totals and surviving coverage, the adversarial verdict, commands and results, blockers, and publication state. Add ticket and optional tool results when applicable, including limitations. Separate safe follow-ups from blockers. Keep the report proportional to the change.
