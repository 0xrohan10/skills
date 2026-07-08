---
name: code-review-loop
description: Open a PR, iterate on Greptile feedback until it reaches 5/5, merge, then monitor and unblock the deployment from main.
---

# Greptile PR Deploy Loop

Use this skill when the user wants the agent to take completed working-tree changes through PR review, Greptile cleanup, merge, and deployment verification.

## Goal

Turn the current branch into a merged PR with a passing deployment from `main`.

The loop is:

1. Open a PR.
2. Wait for Greptile.
3. Fix Greptile comments.
4. Push updates.
5. Repeat until Greptile is 5/5.
6. Merge.
7. Monitor deployment from `main`.
8. Fix deployment blockers on `main` if needed.

## Rules

Do not merge unless:

- Greptile is 5/5.
- Required checks are passing.
- The branch is up to date enough for the repository's merge policy.
- There are no unresolved blocking review comments.
- The PR contains only the intended changes.

Do not force-push unless explicitly requested.

Do not rewrite unrelated code to satisfy vague review feedback.

Do not make cosmetic churn just to appease a reviewer.

Do not commit secrets, generated artifacts, local env files, debug logs, or editor files.

If deployment fails after merge, fix only the blocker needed to restore `main`.

Prefer small follow-up commits over broad rewrites.

## Inputs

- Current working tree
- Current branch
- Repository remote
- Existing test/typecheck/lint commands
- Greptile PR comments and score
- Deployment provider status/logs

## Workflow

### 1. Preflight

Inspect the repo state:

    git status --short
    git branch --show-current
    git diff --stat
    git diff --check

Confirm the branch is not `main` or `master`.

If currently on `main` or `master`, create a new branch before committing:

    git checkout -b <short-descriptive-branch-name>

Inspect the diff before committing:

    git diff --unified=80

Run the narrowest obvious checks before opening the PR.

Prefer:

- targeted tests
- package-level typecheck
- package-level lint
- existing CI-equivalent command if cheap

If checks fail because of the current changes, fix them before opening the PR.

### 2. Commit and push

Stage only relevant files.

Review staged changes:

    git diff --cached --stat
    git diff --cached --check

Commit with a concise message.

Push the branch:

    git push -u origin HEAD

### 3. Open or find the PR

If a PR already exists for this branch, use it.

Otherwise create one.

The PR body should include:

- what changed
- why it changed
- how it was verified
- any known caveats

Keep it short.

### 4. Wait for Greptile

Check whether Greptile has posted a score.

If Greptile has not commented yet:

- sleep 60 seconds
- check again
- repeat until Greptile comments appear

Once Greptile has commented, inspect:

- score
- inline comments
- summary comments
- unresolved threads
- suggested fixes

If Greptile score is less than 5/5:

- sleep 5 minutes before the next full score check
- use the comments to prepare fixes
- apply fixes
- push
- return to the Greptile wait/check loop

Do not treat old Greptile comments as current after pushing a new commit. Wait for the review corresponding to the latest commit.

### 5. Fix Greptile comments

For each comment, classify it as:

- valid blocker
- valid cleanup
- already addressed
- incorrect / not applicable
- risky because it would change behavior

Apply fixes for valid blocker and valid cleanup comments.

For incorrect, stale, or risky comments:

- do not blindly change the code
- leave a concise PR reply explaining why it was not changed, if the harness supports PR comments
- continue the loop and let Greptile re-evaluate

Fix style:

- make the smallest behavior-preserving change
- prefer repository conventions over Greptile's generic preference
- preserve public APIs unless the PR intentionally changes them
- add or update tests only when needed
- do not broaden the scope of the PR

After fixing:

    git status --short
    git diff --stat
    git diff --check

Run relevant checks.

Commit fixes with a concise message.

Push:

    git push

Return to Step 4.

### 6. Exit condition

Exit the review loop only when:

- Greptile score is 5/5 for the latest pushed commit
- required CI checks are green
- no blocking review threads remain

Then merge the PR using the repository's normal merge method.

Prefer the repo's established policy:

- squash merge if that is the default
- merge commit if that is the default
- rebase merge only if that is the default

Do not delete the branch unless that is normal for the repo or the platform does it automatically.

### 7. Monitor deployment from main

After merge, switch to and update `main`:

    git checkout main
    git pull --ff-only

Find the deployment triggered by the merge commit.

Monitor until it reaches a terminal state:

- success
- failure
- cancelled
- timed out

Check:

- deployment status
- build logs
- runtime logs if the build succeeded but health checks failed
- migrations if relevant
- environment variable or secret errors
- dependency or lockfile errors
- typecheck/lint/test failures

### 8. If deployment fails

Fix only the blocker needed to restore deployment from `main`.

Before editing:

    git status --short
    git log --oneline -5

Confirm you are on `main`.

Apply the minimal fix.

Run the narrowest local check that validates the fix.

Commit directly to `main` only when the repo allows direct main commits for deployment unblocks.

If direct main commits are not allowed, create a hotfix branch and PR instead.

Push the fix:

    git push origin main

Monitor deployment again from Step 7.

Repeat until deployment passes.

## Handling uncertainty

Stop and report instead of guessing when:

- Greptile score is unavailable or ambiguous for a long time
- the PR platform cannot be accessed
- required CI is red for reasons unrelated to this PR
- deployment failure appears unrelated to the merge
- fixing deployment requires secrets, infra changes, billing changes, or production data access
- the repository forbids direct commits to `main`
- merge permissions are missing

## Final response

Keep the response short.

Use this format:

    PR:
      <PR title or number>
      <PR URL if available>

    Greptile:
      ✓ 5/5 on latest commit
      - Fixed N comments
      - Skipped M comments with replies

    Merge:
      ✓ Merged into main using <merge method>

    Deployment:
      ✓ Deployment passed
      <deployment URL if available>

    Verification:
      ✓ <command>
      ✓ <command>

    Follow-ups:
      - <only include real follow-ups, or say none>
