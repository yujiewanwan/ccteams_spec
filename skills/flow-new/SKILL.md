---
name: flow-new
description: >
  Start a new spec-driven feature. Triggers on: "/flow:new", "start a new spec",
  "let's spec this out", "new feature", "create a spec for", "I want to build".
  This skill runs the alignment session (PM + Architect + QA agents in parallel)
  and produces the three spec artifacts. Do NOT trigger for "execute", "continue", "sync",
  or any request about an existing spec.
---

# flow:new — Start a New Feature Spec

## What This Skill Does

Runs a structured alignment session with **three agents in parallel**:
- **PM Agent** — product requirements, user stories
- **Architect Agent** — technical feasibility, constraints
- **QA Agent** — testability, edge cases

Parent agent coordinates conflicts and presents a unified draft for human confirmation.

**Output:**
```
docs/specs/<SPEC-ID>/
├── requirement.md    ← human-readable requirements
├── test-cases.md     ← human-readable test cases
└── spec.yaml         ← machine-readable contract
```

Status field drives downstream:
- `draft` → alignment in progress
- `approved` → ready for `/flow:continue`
- `in-progress` → agents working
- `human-testing` → waiting for verification
- `done` → ready for `/flow:sync`

---

## Step 0 — Assign spec_id

Scan `docs/specs/`, increment to next ID (start at SPEC-001 if none).

---

## Stage 1 — Parallel Agent Alignment

Launch three agents simultaneously with the user's input:

**PM Agent** — output: problem, user, success criteria, out-of-scope

**Architect Agent** — output: API changes, data model, constraints, risks

**QA Agent** — output: test approach, edge cases, coverage target

---

## Stage 2 — Conflict Resolution

Parent agent compares outputs:
- Auto-resolve conflicts where possible
- Present remaining conflicts to human in one batch
- Record decisions

---

## Stage 3 — Human Confirmation

Present unified draft:
```
SPEC-XXX Draft
• Problem: ...
• User Story: ...
• Scope: IN: ... / OUT: ...
• Technical: ...
• Test Strategy: ...

Confirm? [yes / edit]
```

---

## Stage 4 — Generate Artifacts

**requirement.md**
```markdown
# Requirement — <title>

spec_id: <SPEC-ID>
status: draft
date: <YYYY-MM-DD>

## Summary
<one sentence>

## Background
<why>

## User Stories

### US-01
**As a** <role>
**I want** <goal>
**So that** <benefit>

## Out of Scope
- <item>

## Open Questions
- <unresolved>
```

**test-cases.md**
```markdown
# Test Cases — <title>

spec_id: <SPEC-ID>
date: <YYYY-MM-DD>
coverage_target: <n>%

## AC-01 — <title>
**Priority:** must | should | nice-to-have
**Type:** unit | integration | e2e | manual
**Automated:** yes | no

**Given:** <given>
**When:** <when>
**Then:** <then>

**Test file:** <path>
**Framework:** <Vitest | Pytest | Playwright | agent-browser>
```

**spec.yaml**
```yaml
spec_id: ""
title: ""
version: "1.0.0"
status: approved
created_at: ""
updated_at: ""
author: ""

git:
  worktree_enabled: false
  worktree_path: ".worktrees/<SPEC-ID>"  # Create .worktrees/ in project root
  branch: ""
  base_branch: "main"
  pr_required: true

alignment:
  product_manager:
    sign_off: true
    notes: ""
  tech_architect:
    sign_off: true
    notes: ""
  qa_manager:
    sign_off: true
    notes: ""
  all_signed_off: true

requirement:
  summary: ""
  background: ""
  user_stories:
    - id: "US-01"
      as_a: ""
      i_want: ""
      so_that: ""
  out_of_scope:
    - ""
  dependencies: []

technical_contract:
  stack:
    frontend: ""
    backend: ""
    database: ""
    other: []
  api_changes:
    - endpoint: ""
      method: ""
      change_type: new
      breaking: false
      request_schema: ""
      response_schema: ""
  data_model_changes:
    - entity: ""
      change_type: new
      description: ""
  non_functional:
    performance: ""
    security: ""
    accessibility: ""

acceptance_criteria:
  - id: "AC-01"
    title: ""
    priority: must
    given: ""
    when: ""
    then: ""
    automated: true
    test_type: unit

agent_teams:
  frontend_engineer:
    scope: ""
    components: []
    api_endpoints_consumed: []
    design_refs: []
    notes: ""
  backend_engineer:
    scope: ""
    services: []
    api_endpoints_owned: []
    database_migrations: false
    notes: ""
  qa_engineer:
    scope: ""
    startup_command: ""
    local_url: ""
    test_mapping:
      - ac_id: "AC-01"
        test_file: ""
        framework: ""
    coverage_target: "80%"
    notes: ""
  architect_reviewer:
    checklist:
      - "API contracts match technical_contract"
      - "No out_of_scope items implemented"
      - "Every AC has a test at the path in test_mapping"
      - "No direct commits to base_branch"
      - "spec.yaml status is approved"
    notes: ""

changelog:
  - version: "1.0.0"
    date: ""
    author: ""
    summary: "Initial spec created."
```

---

## Finish — Human Decision on Worktree

Print artifact summary:
```
docs/specs/<SPEC-ID>/
├── requirement.md    ✓ written
├── test-cases.md     ✓ written
└── spec.yaml         ✓ written  (status: approved)
```

**Assess complexity and ask:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<SPEC-ID> Created
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Complexity Assessment:
• Scope: <frontend/backend/database>
• AC Count: <n>
• Est. Duration: <short|medium|large>

Use worktree for isolated development?
[yes] Use worktree (recommended for complex features, keeps main clean)
[no]  Develop in main directory (quick, no window switching)

Your choice:
```

**If user chooses "yes":**
- Set `git.worktree_enabled: true` in spec.yaml
- Run commit and worktree setup steps below

**If user chooses "no":**
- Set `git.worktree_enabled: false` in spec.yaml
- Print: `Run /flow:continue to start development.`

---

## Worktree Setup (if enabled)

**Check .gitignore for .worktrees/**

```bash
grep -q ".worktrees/" .gitignore 2>/dev/null && echo "Already ignored" || echo "Not found"
```

If not found, ask user:
> `.worktrees/` is not in .gitignore yet. Add it now? [yes/no]

**If user says yes:**
```bash
echo ".worktrees/" >> .gitignore
git add .gitignore
git commit -m "chore: add .worktrees to .gitignore"
```

**Step 1: Commit spec documents to main**

```bash
git add docs/specs/<SPEC-ID>/
git commit -m "<SPEC-ID>: alignment complete [skip ci]"
git tag <SPEC-ID>-alignment
git log --oneline -1
```

**Step 2: Create worktree from the new commit**

```bash
git worktree add .worktrees/<SPEC-ID> -b <branch>
git worktree list
```

Then print:

> ⚠️ **Open a new Claude window for execution.**
> Working directory: `.worktrees/<SPEC-ID>/`
> The spec is at: `docs/specs/<SPEC-ID>/spec.yaml`
> Say: **`/flow:continue`** — Claude will find it automatically.
> Do not run agents in this window.

---

## Finish (if worktree disabled)

```
docs/specs/<SPEC-ID>/
├── requirement.md    ✓ written
├── test-cases.md     ✓ written
└── spec.yaml         ✓ written  (status: approved)

Run /flow:continue to start development.
```
