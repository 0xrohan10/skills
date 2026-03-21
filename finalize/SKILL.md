---
name: finalize
description: "Ship-readiness pass: review, fix, and validate working copy or PR changes. Use when asked to finalize, harden, do a final pass, or make code shippable."
context: fork
---

# Finalize

Your job is to help ship.

Take ownership of the last serious pass before merge: understand what changed, identify what could still go wrong, look for what can be safely simplified, fix everything that has a clear right answer, validate the result, and surface only the decisions that genuinely require product or architectural input.

This is not a generic review skill and not a refactor-for-fun skill. The outcome should be a branch that is safer, simpler, cleaner, better validated, and closer to merge than when you started. Ideally, after you run, the user can click merge and ship. And if something still needs their input, they should have zero confusion about what, why, and what you recommend.

## What "good" looks like

When this skill works well:

- the important risks have been investigated, not guessed at
- the likely blast radius has been mapped and the important affected surfaces have been checked, not assumed safe
- the change has been sized realistically: low, medium, or high risk, with the depth of investigation and validation matched to that level
- any non-trivial finalize edits sit behind a clear rollback boundary, so the user's pre-finalize state is easy to recover
- obvious bugs and regressions are already fixed
- safe, local simplifications have been made where they clearly reduce complexity
- stale docs, broken references, and unresolved review feedback are handled
- validation has actually been run where appropriate, using the highest-fidelity safe checks available for the change
- the checked surfaces and any remaining unverified areas are explicit
- the final report is short, decisive, and actually useful
- the user is left with confidence, not a pile of chores

## Operating stance

Think like the person who would approve this to ship tonight.

- Care most about correctness, regressions, user-visible behavior, failure modes, and maintainability under real team pressure
- Read enough surrounding code to understand the change in context, not just the patch
- Explicitly map the likely blast radius before declaring confidence: callers, dependents, shared utilities, adjacent integrations, affected user flows, tests, docs, and rollout-sensitive surfaces
- Prefer focused investigation over exhaustive ceremony
- Always consider whether the changed code can be made simpler without changing behavior or widening scope
- Use parallel exploration or subagents when helpful, but keep the critical path in the main thread
- If you are running in Claude Code with Agent Teams available, you may use an agent team for broader finalize passes with multiple independent investigations or isolated implementation tracks; prefer simpler subagents when that is enough
- Delegate narrow, concrete questions; do not delegate the whole task
- Optimize for shipping confidence, not for maximum diff size

## Approval boundary

This skill is proactive about getting work ready, not about making irreversible production decisions.

Without explicit user approval in the current conversation, do not:

- deploy anything
- merge anything
- publish or release anything to production
- send external or customer-facing communications
- trigger any other action that would directly affect production users or live systems

You may still prepare the work aggressively:

- fix and simplify the code when the answer is clear
- run validations
- create a branch
- commit task-related changes
- open or update a PR through the normal workflow when safe to do so

If the next step would move from "ready to ship" to "actually shipped," stop and ask.

## Preserve reversibility

Finalize work should be easy to undo.

Before making non-trivial edits of your own, create a clear rollback boundary whenever it is safe to do so. This matters most when you are about to make medium- or high-risk changes, broad simplifications, multi-file rewrites, or deletions of components, helpers, or other structure that may later prove to be useful after all.

Treat the user's pre-finalize state as something worth preserving, not something to overwrite in place.

Default approach:

- if you are on a non-default branch or an existing PR branch and the task-related working copy is coherent, commit the current task state before your larger finalize edits
- then place your own finalize changes in one or more new commits on top, rather than mixing them into that baseline state
- if you are on the default or base branch and the task-related working copy is coherent, create a branch first and then create the baseline commit there before making larger finalize edits
- if the work is already committed and you are only adding finalize edits, keep those edits isolated in their own commit rather than folding them into the existing commit unless the user explicitly asked for that

If the tree is mixed with unrelated edits, the task boundary is unclear, or a checkpoint commit would capture work that should not be bundled together, do not force a commit just to satisfy this rule. In that case:

- avoid invasive simplification or broad rewrites
- either keep your edits narrowly scoped and easy to inspect
- or stop and ask the user before making larger changes that would be hard to disentangle

The goal is not "always commit no matter what." The goal is that any substantial finalize-driven change should usually be reversible with a clean commit boundary, and should almost never be mixed invisibly into an uncommitted working copy.

## Determine the scope

First determine whether the user wants:

- the current working copy
- the current PR
- both

If the user already said which one, use that.

If both a PR and working-copy changes exist and the user did not specify, prefer finalizing the working copy first and mention any PR-only concerns separately. Only stop to ask when the distinction materially changes the work and guessing would be risky.

If there is nothing to finalize, say so plainly.

If the working copy is dirty, be careful about scope hygiene:

- distinguish the changes you are finalizing from unrelated in-progress user edits
- avoid rewriting, reformatting, or "cleaning up" files just because they are already dirty
- do not revert or overwrite user work to make your own edits easier
- when possible, keep your edits confined to the files and hunks that belong to the task you are finalizing
- if the boundary between your work and the user's in-progress changes is unclear and there is real risk of mixing them, stop and ask rather than guessing
- do not make broad or destructive finalize-driven changes in-place unless you first established a safe rollback boundary or the edits are trivially reviewable and reversible

If you are finalizing the current working copy on the repository's default or base branch, treat "ready to ship" as potentially including PR creation, not just code cleanup. If the work ends in high confidence, has no unresolved blockers, and the diff is a coherent task, proactively continue through the repo's normal PR flow:

- create a branch from the finalized working copy
- commit any remaining task-related changes if needed
- create the PR using the project's standard PR workflow, skill, or tooling

Do this only when it is clearly safe. If the tree contains unrelated work, the task boundary is unclear, confidence is not high, or publishing the branch would be premature, stop at the finalize handoff instead of forcing a PR.

## Default execution order

Follow this sequence unless there is a strong reason not to:

1. Determine scope and task boundaries first.
2. Build the picture locally: read the diff, surrounding code, map the likely blast radius, and quickly size the work as low, medium, or high risk before delegating.
3. Decide whether you need a rollback boundary before editing. If your likely changes are non-trivial and the task state is coherent, create the appropriate branch and/or baseline commit first.
4. Decide what must stay on the critical path in the main thread.
5. Spawn parallel subagents, or an agent team when available and justified, only for independent side investigations that can inform the main work without blocking orientation.
6. Synthesize the findings into one concrete view of affected surfaces, likely failure modes, and the fixes or validations that matter.
7. Fix the issues with clear, low-risk answers. Keep cross-cutting or integration-heavy fixes in the main thread, and delegate bounded, isolated implementation tasks when that reduces context load without weakening safety.
8. Validate with the highest-fidelity safe checks available.
9. If the change is medium or high risk, do one final targeted pass over the original blast radius after fixes and validation.
10. Report what was checked, what was fixed, what remains unverified, and why your confidence is what it is.

The user should experience this skill as consistent and predictable: understand first, delegate narrowly, integrate findings, fix decisively, validate realistically, then hand off with a concise decision-oriented report.

## Build the picture fast

Start by getting oriented:

- inspect repository status and the effective diff
- identify changed files and read them in full
- read the relevant project instructions (`AGENTS.md`, `CLAUDE.md`, and referenced docs as needed)
- identify the type of work involved: backend, frontend, infra, schema, integration, refactor, new feature, or a mix
- look beyond the changed files when needed: callers, related tests, docs, routes, schemas, and affected flows

Do not stay trapped inside the diff. A finalize pass is about whether the change holds up in the codebase, not whether the patch looks tidy in isolation.

Before you conclude a change is safe, name the parts of the codebase or product it could plausibly affect and check the ones that matter most. Treat any area with credible impact that you did not inspect as unverified, not implicitly safe.

You do not need a heavyweight artifact, but you should have a clear mental or written impact map before editing: what changed, what it could affect, what looks risky, and what must be validated before calling it ready.

## Investigate what could block shipping

Use your judgment to investigate the likely failure modes for the kind of change you found.

Do not limit this to the edited files. Follow the change through its likely consequences in shared code, downstream consumers, neighboring flows, and operational surfaces where regressions could hide.

Match the depth of the pass to the risk tier. Low-risk changes may need a tight local sweep. Medium-risk changes usually need broader dependency and behavior checks. High-risk changes should trigger a more deliberate pass across affected flows, integration boundaries, and release-readiness concerns.

Examples:

- backend or integration work: input validation, auth, error handling, transaction boundaries, retries/timeouts, status codes, leaked internals, unsafe queries, partial failure behavior
- frontend work: loading/error/empty states, accessibility, responsive behavior, unnecessary effects, stale closures, state ownership, design-system drift, broken interactions
- refactors: missed call sites, stale names, accidental API changes, tests still asserting the old behavior
- new features or greenfield code: missing setup docs, missing env documentation, dev-only assumptions, dependency quality, missing guardrails
- PR work: unresolved review comments or feedback — but read each one critically (see "Handle review feedback with judgment" below)

Use tools and subagents opportunistically. Good delegated work is narrow and concrete:

- conventions and commands
- dependency and impact tracing
- focused backend or frontend audits
- unresolved PR feedback
- targeted verification of a suspicious area

Ask them to return concrete findings with file references and recommended action. Avoid broad summaries that create extra reading without moving the work forward.

If Agent Teams are available in Claude Code, use them only when the task truly benefits from coordinated parallel work across multiple independent investigations or isolated implementation tracks. Do not reach for an agent team for tightly sequential work, same-file edits, or tasks where the coordination overhead would outweigh the gain.

Before you spawn a subagent, know what question it owns, why that question is not the main-thread critical path, and what concrete output you expect back. Good outputs are things like: affected call sites, risky assumptions, missing validations, likely regressions, or a specific surface that appears safe after inspection.

After subagents return, integrate their findings before editing. Do not leave their output as parallel loose ends. The main thread owns the final judgment about blast radius, fixes, and ship confidence.

Once the path is clear, you may also delegate implementation work, but only when the ownership boundary is clean. Good delegated fixes have a concrete goal, a clearly isolated write scope, and low risk of conflicting with other edits or requiring broad architectural judgment.

## Fix aggressively, but only where the answer is clear

Your default is to fix, not to propose.

When the branch is already dirty, be especially disciplined: make the smallest effective edits, preserve unrelated local changes, and avoid broad formatting or mechanical rewrites unless they are required for a safe fix.

Use subagents for implementation only when that makes the work safer or lighter on context, not just to parallelize for its own sake. The main thread should still own cross-file integration, conflict resolution, final review of delegated edits, and any fix where correctness depends on broader codebase judgment.

Treat simplification the same way: always look for it, but only apply it when it is clearly safe and local.

Raise the bar further when a simplification would remove multiple components, delete helper layers, collapse public interfaces, or otherwise erase structure across several files. Those changes are no longer "safe and local" just because they reduce code. Investigate more, checkpoint first, and keep the simplification in its own reversible commit whenever possible.

Good simplification is behavior-preserving and reduces mental load. Typical examples:

- removing dead branches, obsolete flags, or unnecessary indirection
- collapsing duplicated or overly defensive logic when the simplified version is clearly correct
- tightening APIs, names, or state flow when that makes the changed area easier to reason about
- deleting workarounds that no longer serve a purpose

Do not chase elegance for its own sake. If the simplification is speculative, invasive, or likely to trigger unrelated churn, leave it alone.

Fix issues directly when the correct action is clear and low-risk, especially:

- real bugs
- safe simplifications that make the changed area easier to understand or maintain
- broken imports/exports or stale references
- missing obvious edge-case handling
- missing or inconsistent validation
- type-safety holes with a clear local fix
- dead code, commented-out code, debug leftovers, and obvious cleanup
- stale docs or misleading comments
- unresolved review feedback with a clear implementation path — but only when the feedback is actually worth acting on (see below)
- tests that should be updated alongside the change

## Handle review feedback with judgment

Not all review comments deserve implementation. Some are genuinely important. Some are bullshit.

Your job is to tell the difference and act accordingly. Blindly implementing every comment is just as bad as ignoring them all — it wastes effort, bloats the diff, and can make the code worse.

For each unresolved review comment, evaluate whether it:

- **Points to a real bug, risk, or correctness issue** — fix it, this is what finalize is for
- **Suggests a meaningful simplification or catches a genuine oversight** — fix it
- **Is trivial, nitpicky, or stylistic in a way that doesn't matter for shipping** — reply explaining why it's not worth addressing here, and resolve it
- **Is wrong, outdated, or based on a misunderstanding of the code** — reply with the correction, and resolve it
- **Suggests a refactor or improvement that is out of scope for this change** — reply acknowledging it as a valid future improvement if it is one, and resolve it
- **Is vague, generic, or reads like an AI reviewer padding its output** — reply noting that it's not actionable, and resolve it

When you dismiss a comment, be direct and specific about why. "This is a style preference and doesn't affect correctness or readability" is fine. "Not addressing" with no reason is not.

The bar is simple: if implementing the feedback makes the change safer, simpler, or more correct, do it. If it just makes the diff bigger or satisfies a reviewer's taste, skip it and say why.

Do not widen scope just because you noticed something that could be prettier.

Avoid speculative rewrites, style churn, and opportunistic architecture changes unless they are required to make the code safe to ship. "Could be cleaner" is not enough. "Would likely bite us soon" is enough.

## Escalate only real decisions

Flag and ask when a fix depends on product intent, API contract choices, migration strategy, rollout policy, or other decisions you cannot safely infer.

When escalating:

- be specific about the risk
- explain why you did not auto-fix it
- propose the most likely good direction when possible

The user should see a short list of actual decisions, not a hedge against doing the work.

## Validate proportionally

After fixing, run the appropriate checks for the scope of the change.

Default to the smallest sufficient validation set, then expand if risk or failures justify it:

- typecheck for localized code changes
- lint when changes span files or patterns
- targeted tests for the affected area
- build for broader structural changes
- real-flow verification in the best safe environment available when behavior changed: browser, simulator, local app, preview environment, local API calls, CLI exercise, integration harness, or equivalent

If a check fails, treat that as part of finalize: understand it, fix what belongs to this change, and rerun. Do not stop at the first red check unless the failure is unrelated and you can explain that clearly.

Prefer exercising the real behavior of the change, not just reading code or relying on static checks. Use the safest environment that gives meaningful signal: local, test, mock, sandbox, simulator, or preview. Do not use finalize to poke production systems, mutate live data, or trigger irreversible external side effects.

Aim to leave the user with enough practical validation that they would not normally need a second routine testing pass just to establish basic confidence. If that standard was not reached, say so explicitly and explain what remains.

When relevant to the kind of change, also check ship-readiness concerns beyond code correctness: rollout assumptions, migration safety, rollback path, observability, alerting, feature flags, config drift, operational docs, and whether a failure would be detectable and containable. Use judgment here; do not force this checklist onto trivial changes.

## The standard for completion

You are done when one of these is true:

- the work is validated and you would be comfortable shipping it
- the work is validated, you would be comfortable shipping it, and if it started on the default or base branch you have also carried it through the normal branch-and-PR flow when safe to do so
- or there is a short, explicit list of blockers/decisions that genuinely prevent high confidence

If confidence is not high, say exactly why.

## Final response

Keep the final handoff concise and decision-oriented. Include:

- scope reviewed
- risk level and why
- which surfaces or flows you checked, and which remain unverified if any
- what you fixed or simplified
- why those changes were worth making
- what delegated investigations you used and what they established
- what still needs a user decision, if anything
- what you validated directly, what you only inspected, and what remains unverified
- what validation you ran and its result
- whether you created a rollback boundary before editing, and how
- whether you created a branch and PR, and the link if you did
- recommended follow-up improvements or refactors, only if they are clearly valuable and non-blocking
- ship confidence: high, medium, or low, with one short reason

Keep blockers separate from follow-up ideas. The user should be able to scan the response and know both whether this is ready to merge and what would be worth doing next.
