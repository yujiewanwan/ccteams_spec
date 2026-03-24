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

Agent execution is **state-driven** with event-triggered transitions. All agents are launched simultaneously, but QA and Architect start in `waiting` state.

---

**Launch All Agents (Parallel)**

Launch Backend, Frontend, QA, and Architect agents simultaneously:

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

ON COMPLETE:
- Set agent_status.backend_engineer = done (or failed if issues)
- Report summary to parent agent
```

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

ON COMPLETE:
- Set agent_status.frontend_engineer = done (or failed if issues)
- Report summary to parent agent
```

```
## QA Engineer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (acceptance_criteria + agent_teams.qa_engineer)
Read: docs/specs/<SPEC-ID>/test-cases.md

**Current Status:** WAITING for backend and frontend to complete

**Trigger Condition:** Start testing when BOTH backend_engineer = done AND frontend_engineer = done

**Responsibilities:**
1. Auto-discover and start services:
   - Read CLAUDE.md for startup instructions if exists
   - Else detect project type and use default commands:
     * pom.xml/build.gradle → ./mvnw spring-boot:run / ./gradlew bootRun
     * package.json → npm run dev / npm start
     * requirements.txt/pyproject.toml → python main.py / uvicorn
     * go.mod → go run .
     * Cargo.toml → cargo run
   - Start backend and frontend, wait for <local_url> ready

2. Run ALL tests:
   - unit tests
   - integration tests (against running services)
   - e2e tests with agent-browser (real browser, automated)
     - MUST use agent-browser tool, never mock
     - Include browser steps as comments

3. Stop services after testing

**Feedback Loop:**
- If tests FAIL: Set agent_status.qa_engineer = failed, report failing ACs to parent
- Parent will re-dispatch Backend/Frontend with fix instructions

Coverage target: <coverage_target>

Hard constraints:
- YOU start the services, do not ask human
- Every "must" AC must have a passing test
- Use agent-browser for e2e, never mock browser
- Do not test anything in out_of_scope

ON COMPLETE (all tests pass):
- Set agent_status.qa_engineer = done
- Report test results to parent agent
```

```
## Architect Reviewer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (full)

**Current Status:** WAITING for QA testing to complete

**Trigger Condition:** Start review when qa_engineer = done

Review checklist:
- API contracts match technical_contract
- No out_of_scope items implemented
- Code quality acceptable
- All tests pass
- Architecture decisions followed

For each failure: comment with AC id, set status = failed, return to relevant agent.

ON COMPLETE:
- Set agent_status.architect_reviewer = done (or failed if issues)
- Report review results to parent agent
```

---

**State Transitions**

```
Initial:
  backend_engineer: in-progress
  frontend_engineer: in-progress
  qa_engineer: waiting
  architect_reviewer: waiting

When backend_engineer = done AND frontend_engineer = done:
  → Set qa_engineer = in-progress (trigger QA to start testing)

When qa_engineer = done:
  → Set architect_reviewer = in-progress (trigger Architect review)

When qa_engineer = failed:
  → Parent agent analyzes failures
  → Re-dispatch affected agents (backend or frontend) with fix instructions
  → Reset qa_engineer = waiting
  → Loop until all tests pass

When ALL agents = done:
  → Proceed to Pre-test Gate
```

---

**Parent Agent Coordination Logic**

Parent agent must:
1. Launch all 4 agents simultaneously
2. Monitor agent_status changes
3. Trigger state transitions based on completion events
4. Handle failure feedback loops

Example coordination output:
```
SPEC-002 Agent Execution
─────────────────────────────────────────
[◐] Backend Engineer    in-progress
[◐] Frontend Engineer   in-progress
[○] QA Engineer         waiting (for backend+frontend)
[○] Architect Reviewer  waiting (for QA)
─────────────────────────────────────────
```

When Backend and Frontend done:
```
SPEC-002 Agent Execution
─────────────────────────────────────────
[✓] Backend Engineer    done
[✓] Frontend Engineer   done
[◐] QA Engineer         in-progress  ← triggered
[○] Architect Reviewer  waiting
─────────────────────────────────────────
```

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

**QA Engineer (starts services + tests with agent-browser):**
```
## QA Engineer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (acceptance_criteria + agent_teams.qa_engineer)
Read: docs/specs/<SPEC-ID>/test-cases.md

Responsibilities:
1. Start services automatically (do NOT wait for human):
   - Backend: <startup_command_backend> (e.g., ./mvnw spring-boot:run)
   - Frontend: <startup_command_frontend> (e.g., npm run dev)
   - Wait for <local_url> to be ready

2. Run ALL tests:
   - unit tests
   - integration tests (against running backend)
   - e2e tests with agent-browser (real browser, automated)
     - MUST use agent-browser tool, never mock
     - Include browser steps as comments

3. Stop services after testing

Coverage target: <coverage_target>

Hard constraints:
- YOU start the services, do not ask human to start
- Every "must" AC must have a passing test
- Use agent-browser for e2e, never mock browser
- Do not test anything in out_of_scope
```

**Architect Reviewer:**
```
## Architect Reviewer — <spec_id>

Read: docs/specs/<SPEC-ID>/spec.yaml (full)
Read: code changes from Backend/Frontend agents

Review checklist:
<architect_reviewer.checklist items>
- API contracts match technical_contract
- No out_of_scope items implemented
- Code quality acceptable

For each failure: comment with AC id, return to relevant agent for fix.
```

---

### If status = `in-progress` → Verify Agent Completion

Check `agent_status`:
- ALL agents should be `done`
- If any agent is `failed` → re-dispatch that agent with fix instructions
- If QA agent reported test failures → set backend/frontend status back to `pending` and re-run Phase 1-2

Once all agents `done`:
- Set `status: human-testing` in spec.yaml
- Run human-test step below

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

**Calling `/flow:continue` when status is `human-testing` means the user has returned from manual testing.**

Treat this as "testing complete" by default. Print:

```
Human Test Checklist — <spec_id>
─────────────────────────────────────
[ ] AC-01 — <title>
[ ] AC-02 — <title>
─────────────────────────────────────
```

Then ask (one question, two options):
> All manual tests passed?
> [yes — mark done] / [no — describe what failed]

**User says "yes" or "done" or "passed":**
- Set `status: done` in spec.yaml
- Print:
  ```
  ✓ Marked done. Run /flow:sync to update docs and generate PR checklist.
  ```

**User reports failures:**
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
