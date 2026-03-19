---
name: flow-continue
description: >
  Resume a spec in progress. Triggers on: "/flow:continue", "continue",
  "next step", "what's next", "resume spec", "keep going".
  This skill reads spec.yaml status and automatically determines the next
  action — no need to remember which phase you were in.
  Works after human-test interruptions, context resets, or window switches.
---

# flow:continue — Resume From Where You Left Off

## What This Skill Does

Reads `spec.yaml` status and executes exactly the next required action.
One step at a time. Safe to call repeatedly — each call advances one step.

---

## Step 1 — Find the Spec

Scan `docs/specs/` in the current working directory:
- Collect every subdirectory containing a `spec.yaml`
- **One found** → use it, print confirmation, continue
- **Multiple found** → list them, ask which one:
  ```
  Found multiple specs:
    SPEC-001 — User Login (status: approved)
    SPEC-002 — Password Reset (status: draft)
  Which spec should I continue?
  ```
- **None found** → report error, suggest running `/flow:new`

Always confirm before acting:
> **Continuing:** SPEC-001 — "User Login" (status: approved)

---

## Step 2 — Read Status and Print Artifact State

Read `spec.yaml` status field. Print a state summary:

```
SPEC-001 — User Login
─────────────────────────────────────────
✓ requirement.md      done
✓ test-cases.md       done
✓ spec.yaml           approved
◆ agent execution     ready
○ pre-test gate       blocked (needs: agent execution)
○ human-test          blocked (needs: pre-test gate)
○ sync                blocked (needs: human-test)
─────────────────────────────────────────
Next: agent execution
```

Status → artifact state mapping:

| spec.yaml status | Artifact state display |
|---|---|
| `draft` | alignment incomplete — run `/flow:new` to finish |
| `approved` | agent execution = **ready** |
| `in-progress` | agent execution = done, pre-test gate = **ready** |
| `human-testing` | pre-test gate = done, human-test = **ready** |
| `done` | all done — run `/flow:sync` |
| `deprecated` | spec retired, no action |

---

## Step 3 — Execute the Next Ready Step

### If status = `approved` → Run Agent Execution

Set `status: in-progress` in spec.yaml before starting.

Dispatch each agent with only their relevant spec sections.
Agents run in this order: Backend → Frontend → QA (in parallel where possible).

**Backend Engineer:**
```
## Backend Engineer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (technical_contract + agent_teams.backend_engineer)

Scope: <backend_engineer.scope>
Services: <services>
Endpoints to implement: <api_endpoints_owned>
Data model changes: <data_model_changes>
Migration required: <database_migrations>
Non-functional: <non_functional>
Acceptance criteria: <AC items where test_type is integration>

Hard constraints:
- Do not implement anything in requirement.out_of_scope
- Never store plaintext credentials
- No direct commits to <git.base_branch>
```

**Frontend Engineer:**
```
## Frontend Engineer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (agent_teams.frontend_engineer)

Scope: <frontend_engineer.scope>
Components: <components>
API endpoints to consume: <api_endpoints_consumed>
Design refs: <design_refs>
Acceptance criteria: <AC items where test_type is e2e or manual>

Hard constraints:
- Do not implement anything in requirement.out_of_scope
- Do not add any UI elements not listed in components
```

**QA Engineer:**
```
## QA Engineer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (acceptance_criteria + agent_teams.qa_engineer)
Read: docs/specs/<SPEC-ID>/test-cases.md

Write and run all tests per test_mapping.
Coverage target: <coverage_target>

Tooling by test_type:
- unit        → <project unit framework>
- integration → Supertest / httpx against local server
- e2e         → agent-browser (real browser, not mocked)
  Start service first: <startup_command>
  Test against: <local_url>

Hard constraints:
- Every "must" AC must have a passing test — no exceptions
- e2e test files must include browser steps as comments for reproducibility
- Do not test anything in out_of_scope
```

**Architect Reviewer:**
```
## Architect Reviewer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (full)
Read: full PR diff

Checklist — do not approve until all true:
<architect_reviewer.checklist items>

For each failure: comment with the AC id or out_of_scope item.
Return to the relevant agent with a specific fix instruction.
```

---

### If status = `in-progress` → Run Pre-Test Gate

The QA agent must confirm all of the following before human-test:

```
Pre-test gate — <spec_id>

[ ] Local service running at <local_url> (start: <startup_command>)
[ ] All unit tests passed
[ ] All integration tests passed
[ ] agent-browser e2e tests passed
[ ] No console errors on pages covered by e2e ACs
[ ] Coverage meets or exceeds <coverage_target>
```

If all green → set `status: human-testing` in spec.yaml, then run the human-test step below.
If any red → fix and re-run. Do not advance status.

**Human-test checklist** (print after gate passes):

Print only ACs where `test_type: manual` or `automated: false`:

```
Human Test Checklist — <spec_id>
Service running at: <local_url>

[ ] AC-XX — <title>
    Given: <given>
    When:  <when>
    Then:  <then>

Report each as PASS or FAIL. For FAIL, describe what you saw.
```

Then stop and wait. The user tests manually.
When they return and say `/flow:continue` or "done testing", proceed to next status.

---

### If status = `human-testing` → Confirm Completion

Print human test checklist (for reference only):

```
Human Test Checklist — <spec_id>
─────────────────────────────────────
[ ] AC-01 — <title>
[ ] AC-02 — <title>
─────────────────────────────────────
```

Then ask:
> Is human testing complete? I will proceed to the final stage.
> [complete / issues found]

**User replies "complete" or "done":**
- Set `status: done` in spec.yaml
- Print:
  ```
  Confirmed complete. Run /flow:sync to update docs and generate PR checklist.
  ```

**User reports issues:**
- Ask for details
- Record `failure_note` on the failing AC in spec.yaml
- Set `status: in-progress`
- Re-dispatch only the relevant agent(s) with the failure description
- Print:
  ```
  Issues recorded. Re-dispatching affected agents.
  Run /flow:continue when fixes are ready.
  ```

---

### If status = `done` → Remind to Sync

```
This spec is done. Run /flow:sync to update all artifacts and get the PR checklist.
```

---

### If status = `draft` → Alignment Incomplete

```
Alignment not complete (status: draft).
Run /flow:new to finish the alignment session.
```

---

## Hard Rules

1. Never start agent execution if `all_signed_off: false`.
2. Never advance status without completing the current step's gate.
3. Never assume — if spec.yaml is ambiguous, ask before acting.
4. Always print the artifact state summary at the start of every `/flow:continue` call.
