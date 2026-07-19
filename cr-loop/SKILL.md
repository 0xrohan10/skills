---
name: code-review-loop
description: "Run an explicitly requested PR-to-deployment loop: publish a PR, reach Greptile 5/5, merge, and verify deployment. Use only when the user asks for the full lifecycle."
---

# Code Review Loop

Own only the explicitly requested PR-to-deployment lifecycle. Start from a merge-ready tree produced by `finalize` or equivalent evidence; general hardening belongs to `finalize`, and behavior-preserving cleanup belongs to `simplify`.

## Authorization and Outcomes

Start when the current conversation explicitly authorizes the full lifecycle: publish the branch, open or update the PR, handle review feedback, merge after the gate passes, and monitor and recover the resulting deployment. Missing authorization produces `BLOCKED_AUTHORIZATION` before the first ungranted transition.

The terminal outcomes are:

- `DEPLOYED`: every merge and deployment gate passed.
- `BLOCKED_PREFLIGHT`: the tree, repository state, base branch, or required access is not ready.
- `BLOCKED_REVIEW`: the latest head SHA cannot reach the review gate within its budget.
- `BLOCKED_MERGE`: the review gate passed but policy, permissions, or platform state prevents a verified merge.
- `BLOCKED_DEPLOYMENT`: deployment cannot pass within the recovery budget or requires an unsafe/unauthorized decision.

Use normal additive commits and pushes. A force push, direct base-branch commit, production-data mutation, secret/permission change, billing change, or infrastructure policy change requires separate explicit authorization and repository support.

## Resolve Invariants

Before changing repository state:

1. Read applicable repository instructions and discover commit, PR, merge, required-check, and deployment policies.
2. Resolve `<base-branch>` from the user's explicit target when supplied; an existing PR base must match it. With no explicit target, use the existing PR base, then the remote default from provider metadata or `origin/HEAD`. Verify the branch exists remotely, record it once, and use `<base-branch>` throughout.
3. Resolve the topic branch, remote, PR platform, required reviewers/checks, merge method, deployment provider, and required deployment targets.
4. Record finite budgets. Use repository/user values when supplied; otherwise use five reviewed head SHAs per PR, 20 minutes of review polling per head SHA at intervals of at least 60 seconds, two deployment-recovery attempts, and 30 minutes per required deployment.

Invariant resolution is complete when every placeholder above has one evidenced value and every budget is finite. Conflicting or missing material values produce `BLOCKED_PREFLIGHT`.

## Preflight and Publish

Require a `TREE_MERGE_READY` result for the current tree or equivalent evidence covering scope, findings, simplification, and validation. Otherwise return `BLOCKED_PREFLIGHT` and route preparation to `finalize`.

Inspect current status, branch, full intended diff, untracked files, and `git diff --check`. Confirm that unrelated work is excluded from staging and that intended generated artifacts, secrets policy, and repository checks still match the readiness evidence.

When the current branch is `<base-branch>`, create the authorized topic branch before staging. Stage only intended files, inspect the staged diff and check it, commit under repository policy, and push normally. Reuse an existing PR for the topic branch or open one with the change, rationale, validation, and material caveats.

Preflight is complete when the remote topic head equals local `HEAD`, the PR targets `<base-branch>`, and the PR diff contains exactly the intended changes.

## Bounded Head-SHA Review

Each PR gets an independent head-SHA budget. A cycle is bound to one immutable `<head-sha>`:

1. Capture the remote PR head SHA after each push and increment the reviewed-head count. A SHA beyond the configured count produces `BLOCKED_REVIEW`.
2. Poll until the per-SHA deadline for Greptile output and required checks that platform evidence ties to that exact SHA. Older comments and scores remain history, not current evidence.
3. Record the Greptile score, required-check results, blocking threads, and a disposition for every current comment: fix, already addressed, incorrect/outdated, out of scope, or behavior-risk decision.
4. Route cleanup-only fixes through `simplify` with the PR's resolved `<base-branch>` and current `<head-sha>` as its effective diff. Apply correctness fixes only when repository evidence gives a clear in-scope answer; validate every invalidated surface. Reply with evidence for no-change dispositions.
5. Commit and push one coherent correction set. The new remote SHA begins the next cycle; the prior cycle cannot satisfy its gate.

A behavior-risk decision that needs product or contract intent, a required check failing for an unrelated cause, unavailable review evidence at the deadline, or an exhausted head budget produces `BLOCKED_REVIEW` with the exact evidence.

## Merge Gate

Merge only when all statements are true at the same observed `<head-sha>`:

- `<head-sha>` equals the latest remote PR head.
- Greptile reports 5/5 for `<head-sha>`.
- Every required check reports success for `<head-sha>`.
- Every blocking thread is resolved and every review comment has a recorded disposition.
- The PR is mergeable and satisfies the repository's update and approval policy.
- A fresh comparison against `<base-branch>` contains exactly the intended diff.

Use the repository's established merge method. Capture `<merged-sha>` from the platform, then verify that remote `<base-branch>` contains it or, for squash/rebase workflows, contains the platform-reported resulting commit. Failure of any statement produces `BLOCKED_MERGE`; review failures return to the bounded review loop only while budget remains.

## Deployment Gate

Locate each required deployment triggered by the resulting `<merged-sha>` or by a later `<base-branch>` commit proven to contain it. Monitor each until success or its deadline, retaining provider status and relevant build, migration, runtime, and health evidence.

Report `DEPLOYED` only when all statements are true:

- The PR is merged into the resolved `<base-branch>` and remote history contains the platform-reported result.
- Every required deployment is tied to that result or a proven descendant on `<base-branch>`.
- Every required deployment and post-deploy health/release check reports terminal success within its deadline.
- No required target remains pending, failed, cancelled, or timed out.

## Bounded Deployment Recovery

When a required deployment fails, diagnose the smallest causal blocker from provider evidence. An unrelated failure, unclear causality, or a fix requiring a separately authorized action produces `BLOCKED_DEPLOYMENT`.

For each available recovery attempt:

1. Increment the recovery count, update remote `<base-branch>`, and create a recovery branch from that exact tip. The default recovery path is branch plus PR; the base branch remains an integration target rather than an editing workspace.
2. Apply only the causal fix and run `finalize` to establish `TREE_MERGE_READY` for the recovery diff.
3. Publish a recovery PR targeting `<base-branch>`. Give it a fresh bounded head-SHA review loop and require the complete merge gate.
4. Capture the new merge result and rerun the complete deployment gate for every invalidated target.

Use a direct `<base-branch>` commit only when the user separately authorizes it in the current conversation and repository policy permits it. Exhausting the recovery count or any recovery PR's review budget produces `BLOCKED_DEPLOYMENT`.

## Report

Return the terminal outcome; `<base-branch>`; PR URL; latest reviewed head SHA and budget usage; Greptile score, checks, and comment dispositions; merge method and resulting SHA; deployment targets, URLs, and evidence; recovery usage; and every blocker. Claim success only as `DEPLOYED` after the complete deployment gate passes.
