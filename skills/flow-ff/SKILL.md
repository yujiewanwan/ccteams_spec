---
name: flow-ff
description: >
  Fast-forward a spec through all remaining steps without stopping.
  Triggers on: "/flow:ff", "fast forward", "run everything", "skip to the end",
  "just do it all". Use when the spec is simple and you trust the agents to
  run without step-by-step confirmation. Skips human-test phase — only use
  when all ACs are automated.
---

# flow:ff — Fast-Forward All Steps

## What This Skill Does

Runs all remaining steps from current status to `done` without pausing for
confirmation between steps. Equivalent to calling `/flow:continue` repeatedly.

**Only safe when:**
- All acceptance criteria have `automated: true`
- No `test_type: manual` ACs exist
- You have reviewed the spec and trust the agent output

If any `manual` ACs exist, stop at `human-testing` and tell the user to
run `/flow:continue` after they complete manual testing.

---

## Step 1 — Find and Confirm Spec

Same scan logic as `/flow:continue`. Print confirmation:
> **Fast-forwarding:** SPEC-001 — "User Login" (status: approved)

---

## Step 2 — Check for Manual ACs

Read `acceptance_criteria` in spec.yaml.
If any AC has `test_type: manual` or `automated: false`:

```
⚠️  Manual ACs detected:
    AC-06 — Error state visual (manual)

/flow:ff will pause at human-testing for these items.
Proceeding with automated steps now...
```

---

## Step 3 — Run All Steps in Sequence

Print a live progress list and execute each step:

```
SPEC-001 fast-forward
──────────────────────────────────
[ ] Agent execution
[ ] Pre-test gate
[ ] Human-test (automated ACs only)
[ ] Sync artifacts
──────────────────────────────────
```

Execute each step using the same logic as `/flow:continue`.
Update the checkbox as each step completes:

```
SPEC-001 fast-forward
──────────────────────────────────
[✓] Agent execution
[✓] Pre-test gate
[✓] Human-test (all automated)
[✓] Sync artifacts
──────────────────────────────────
Done. PR checklist printed below.
```

---

## Step 4 — If Manual ACs Exist, Pause at human-testing

When pre-test gate passes and manual ACs remain:

```
SPEC-001 fast-forward
──────────────────────────────────
[✓] Agent execution
[✓] Pre-test gate
[⏸] Human-test — manual steps required

Human Test Checklist:
[ ] AC-06 — Error state is visually clear
    Given: submit form with wrong credentials
    When:  API returns 401
    Then:  inline error appears; button re-enables

Complete these steps, then run /flow:continue.
──────────────────────────────────
```

Stop here. Do not advance to sync.

---

## Step 5 — After All Steps Complete

If no manual ACs (or after `/flow:continue` processes manual results):
Run `/flow:sync` logic inline — update all three artifacts, **commit, create PR**, and print summary.

```
All steps complete.

PR created: <URL>
──────────────────────────────────
[✓] Agent execution
[✓] Pre-test gate
[✓] Human-test (all automated)
[✓] Sync artifacts + commit + create PR
──────────────────────────────────
```
