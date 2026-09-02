---
name: test-value-audit
description: Audit test value. Use when the user asks to review tests for regression value or prune low-value unit and integration tests.
---

# Test Value Audit

Triage every in-scope test as **KEEP**, **MERGE**, **REWRITE**, or **DELETE**. Preserve regression signal while reducing maintenance noise.

## Modes

Use the tests, module, or suite named by the user as the audit boundary.

- A review request produces findings only.
- A cleanup request authorizes edits within that boundary.
- Skill invocation without a cleanup request uses review mode.

## Procedure

### 1. Establish the baseline

Read the in-scope tests, the production code they exercise, neighboring unit and integration tests, and any comments or issue references that explain regression history. Run the relevant test command before editing.

Complete this step when every in-scope test is mapped to its production behavior and neighboring coverage, and the baseline command and result are recorded.

### 2. Triage every test

Before classifying tests, read [`EVIDENCE.md`](EVIDENCE.md) in full. Apply its deletion gate, evidence patterns, seam checks, and protected-contract rules to every in-scope test.

For each test, identify:

- The observable contract it protects.
- A plausible regression it would detect.
- Other tests that detect the same regression.
- Its distinct diagnostic value.

Assign one decision:

- **KEEP**: distinct, useful regression protection.
- **MERGE**: useful protection spread redundantly across tests.
- **REWRITE**: an important scenario that the current assertions do not protect.
- **DELETE**: no meaningful protection beyond surviving tests.

Complete this step when every in-scope test has one decision and supporting evidence. Every **DELETE** decision must name a surviving replacement test by file and test name and explain which assertion detects the same regression. Classify unresolved contracts as requiring human judgment.

### 3. Apply authorized changes

In cleanup mode, change tests in small groups that have independent replacement coverage. Keep production code unchanged. Tests in the same deletion group cannot serve as replacements for one another.

Complete each group when every edit matches a triage decision and all named replacement tests remain in the suite.

### 4. Verify each group

Run the relevant tests after each group. Use available coverage tooling to inspect meaningful coverage changes, while treating executed lines as supporting data rather than replacement evidence.

If a command fails, separate baseline failures from failures introduced by the group. Repair or revert only the edits responsible for new failures before continuing.

Complete this step when every changed group has a recorded post-change result and no unresolved failure caused by the audit.

### 5. Report the audit

Report every in-scope test:

| Test | Decision | Protected behavior | Value evidence | Surviving replacement | Confidence |
| --- | --- | --- | --- | --- | --- |

For each deletion, include:

> A regression that changes ___ would still be detected by ___ because ___.

Finish with totals for tests deleted, merged, rewritten, retained despite apparent overlap, and requiring human judgment. List commands and results, plus meaningful coverage changes.

Complete the audit when the table accounts for every in-scope test, every deletion names concrete surviving coverage, and all verification results are reported accurately.
