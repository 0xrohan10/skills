---
name: adversarial-code-review
description: Adversarial code review that tries to disprove a change is safe. Produces counterexamples, not vibes. Use when the user wants adversarial, hostile, or red-team review, or asks you to break, attack, disprove, or exploit the change.
---

# Adversarial Code Review

**Disprove that the change is safe.** Do not confirm that it looks reasonable.

Posture: *assume the implementation is subtly wrong — what evidence would expose it?* Adversarial toward the **code and its assumptions**, never the author.

Read-only on source. Findings only — never edit, commit, or fix while reviewing.

## Scope

1. Resolve the evidence source: working tree by default, or the base/head the user names. Read status, full diff and stat, and hand-authored untracked files in scope.
2. Read the system around the diff: callers, callees, DB constraints and migrations, retry policy, auth model, queue semantics, deployment/rollback order, observability, and older versions that may coexist.
3. State the **claimed invariants** the change must preserve (money conserved, authz before access, no duplicate side effects, exclusive reservation, internally valid persisted state, …). If neither the PR nor the code can name them, that is already a finding.

Scope is done when every in-scope file is listed, every claimed invariant is written down, and the surrounding system surfaces above have been opened. Pre-existing debt counts only when the change widens its **blast radius**.

## What to Attack

Work outward from the change. Normal review follows control flow; adversarial review **interrupts** it.

### 1. Claimed invariants

For each invariant: what enforces it — code path, DB constraint, lock, type, or test? If nothing does, construct a **counterexample**.

### 2. Hidden assumptions

Hunt assumptions true on the happy path but unenforced:

- once-and-in-order delivery
- read stays valid until the next write
- external responses are well-formed
- clocks agree; IDs are globally unique
- one tenant per user; "optional" fields are present
- the process cannot die between two statements

The strongest findings sound like: *this works only if X, and nothing enforces X.*

### 3. Failure boundaries

Stop execution at every seam:

- before the transaction / halfway through / after commit before ack
- during a retry
- after one external side effect but before another
- after schema deploy before all app instances update

### 4. Concurrency and duplication

Run the operation twice, simultaneously, out of order, against stale state, and after a timeout where the caller does not know whether it succeeded. Expose missing uniqueness, weak idempotency, read-modify-write races, wrong lock scope.

### 5. Trust boundaries

Treat every value crossing a boundary as malicious or malformed: client input, webhooks, queue messages, rows written by older code, config, third-party responses, filenames, URLs, serialized objects. Validation belongs where trust changes, not only where data first enters.

### 6. The tests

Do not admire coverage. Ask whether the suite could catch the bugs above. Prefer: invariant/property tests, two concurrent executions, duplicate delivery, crash between side effects, malformed-but-type-valid input, boundary values, partial third-party failure, old-schema/new-code and new-schema/old-code. A test that mirrors the implementation only proves the author wrote the same assumption twice.

## Process

1. **Name invariants** — write them explicitly. Done when each has a one-line statement and a candidate enforcer (or "unenforced").
2. **Build counterexamples** — for each invariant and attack axis above, construct the strongest plausible execution that breaks it. Read through constraints, callers, and retries until the counterexample is confirmed or killed. Done when every axis has a confirmed finding or a *held* note naming what rules it out.
3. **Rank** — keep only real counterexamples. Severity = probability × impact × detectability. Drop style, taste, aesthetic abstraction demands, negligible hypotheticals, and personal preference.
4. **Verdict**.

## Output

### Findings

One table. One row per finding. No praise section. No low/nit tier — if it is not a break, it is not a finding.

| Severity | Location | Invariant | Counterexample | Impact | Enforcement |
| --- | --- | --- | --- | --- | --- |
| critical \| high \| medium | `file:line` | what must stay true | concrete sequence (actors, interleaving, inputs) | blast radius | smallest mechanism that rules it out |

Every finding is a **counterexample**, not anxiety. Four parts, always: violated invariant, concrete sequence, impact, proposed enforcement.

Example shape:

> Two workers can settle the same invoice because idempotency is checked outside the transaction. Both read no settlement, both submit the transfer, only afterward record completion — double-pay. Put the idempotency claim behind a unique constraint and acquire it before the external transfer.

- **critical** — concrete path to data loss/corruption, auth bypass, secret exposure, or cross-tenant breach
- **high** — exploitable with common privileges or realistic races; material integrity/confidentiality loss
- **medium** — real break under tighter preconditions, or limited blast radius

### Verdict

A review succeeds when it does one of:

1. Finds a concrete correctness or security failure → **Block**
2. Raises confidence by showing important failure modes are already prevented → **Approve**, with those modes listed as held
3. Forces an implicit invariant into the open (code, constraint, test, or docs) → **Block** until it is explicit, unless the only gap is documentation and the enforcement already holds

- **Block** — any critical/high counterexample, any unenforced critical invariant, or any confirmed escalation below.
- **Approve** — every attack axis was probed, every critical invariant has a named enforcer, no confirmed counterexample remains. Residual risks listed with why they do not block.

### Hard escalations

Flag on sight:

- Unenforced invariant over money, authz, tenancy, or durability
- User-controlled value reaches a sink without a proven guard
- Secret or credential material in source, logs, or fixtures the diff touches
- Side effect issued before the idempotency/lock claim is durable
- Trust moved client-side or onto an unverified webhook/callback
- Catch-and-ignore on a security or integrity path
- Prior guard weakened or removed (validation, uniqueness, rate limit, sandbox)

## Not This

Adversarial review is not maximizing comment count, blocking on taste, inventing negligible hypotheticals, asking the author to prove facts the reviewer can inspect, or turning every PR into an architecture referendum.
