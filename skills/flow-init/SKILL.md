---
name: flow-init
description: >
  Bootstrap a spec from existing documentation or context.
  Triggers on: "/flow:init", "init from", "bootstrap spec", "convert to spec",
  "start from plan", "make this a spec", "flow from".
  Use when you ALREADY have a plan, requirements doc, or notes that you want
  to convert into the flow-spec format and start developing immediately.
  Do NOT trigger when starting from scratch — use /flow:new for that.
---

# flow:init — Bootstrap from Existing Documentation

## What This Skill Does

Converts **existing documentation** into a complete flow-spec with one command.
Unlike `/flow:new`, this skips the three-agent alignment debate — you already
did the thinking, now let's execute.

**Key principle:** Preserve provenance. Always reference or embed the source
document in `requirement.md` so the team knows where this spec came from.

---

## Usage Patterns

```bash
# Pattern 1: Auto-detect context (recent plan, docs/plan.md, etc.)
/flow:init

# Pattern 2: Explicit file path
/flow:init from docs/plan/login-redesign.md
/flow:init from notion://Login-Page-Redesign

# Pattern 3: Raw text (for quick MVPs)
/flow:init "Add a dark mode toggle to the navbar"

# Pattern 4: Multiple sources
/flow:init from docs/requirements.md + docs/api-contract.yaml
```

---

## Step 1 — Detect Input Source

Scan in this priority order:

### 1a. Check explicit argument
- User said `from <path>` → use that
- User provided quoted text → treat as raw requirement

### 1b. Check conversation context
- Recent `/plan` output in the same thread
- Recent pasted content that looks like a spec (has sections, acceptance criteria, etc.)

### 1c. Check common locations
- `docs/plan*.md`
- `docs/requirement*.md`
- `plan.md` in root
- `CLAUDE.md` (if it contains a plan section)

### 1d. Ask user
```
What should I use as the source?
[recent plan in context] / [path: _____] / [paste content]
```

### Source metadata to capture
```yaml
source:
  type: file | context | text | url
  path: <original path or null>
  title: <extracted title or "Untitled Plan">
  date: <file mtime or today>
  author: <git author of file or user>
```

---

## Step 2 — Validate and Parse Source

### If source is a file
- Verify file exists
- Read full content
- Detect format: markdown plan, structured spec, free text
- Extract key sections if present: problem, solution, acceptance criteria, API changes

### If source is conversation context
- Extract the plan section from recent messages
- Preserve as-is

### If source is raw text
- Treat as `summary` field
- Flag that ACs need to be generated

### Validation errors
```
⚠️  Cannot read source: <path> not found
Please check the path or paste the content directly.
```

---

## Step 3 — Generate Spec (Single Agent Mode)

Launch ONE agent (Spec Bootstrap Agent) with:

```
## Spec Bootstrap Agent

Read: <source content>

Your task:
1. Analyze the source and extract all structured information
2. Generate the three spec artifacts below
3. Ensure acceptance criteria are testable
4. Infer missing fields with reasonable defaults

Source metadata:
  type: <source.type>
  path: <source.path>
  title: <source.title>

Output format:

**requirement.md**
- Include "## Source" section with full provenance
- Embed or reference the original document
- Structure: Summary, Background, User Stories, Out of Scope, Source

**test-cases.md**
- One AC per acceptance criteria from source
- Generate testable Given/When/Then if missing
- Mark test_type: unit|integration|e2e|manual based on context

**spec.yaml**
- status: approved (skip draft phase)
- Copy all technical details from source
- Set agent_teams with appropriate scope hints
- agent_status all pending except qa/architect blocked
```

---

## Step 4 — Assign Spec ID

Same logic as `/flow:new`:
- Scan `docs/specs/` for existing IDs
- Increment to next (SPEC-001 if none)

---

## Step 5 — Write Artifacts

### requirement.md structure (with source preservation)

```markdown
# Requirement — <title>

spec_id: <SPEC-ID>
status: approved
date: <YYYY-MM-DD>

## Summary
<extracted or generated summary>

## Background
<extracted background>

## User Stories

### US-01
**As a** <role>
**I want** <goal>
**So that** <benefit>

## Out of Scope
- <item>

## Source

<!-- This spec was bootstrapped from existing documentation -->

**Original Source:** `<source.path>`
**Type:** <source.type>
**Date:** <source.date>
**Author:** <source.author>

### Full Original Content

<details>
<summary>Click to expand original document</summary>

---
<embed full source content here>
---

</details>

### Attribution Notes
- This spec was auto-generated via `/flow:init`
- Original document preserved above for reference
- Any discrepancies should be resolved in this spec (not the source)
```

### test-cases.md structure

```markdown
# Test Cases — <title>

spec_id: <SPEC-ID>
date: <YYYY-MM-DD>
coverage_target: 80%

## AC-01 — <title>
**Priority:** must
**Type:** <inferred from context>
**Automated:** <yes if testable, no if manual>

**Given:** <given>
**When:** <when>
**Then:** <then>

<!--
Source: Extracted from <source.path>
Original text: "..."
-->
```

### spec.yaml structure

Same as `/flow:new` output, but:
- `status: approved` (not draft)
- `alignment.all_signed_off: true` (bypass alignment)
- `provenance.source` section added:

```yaml
provenance:
  bootstrapped_from:
    type: file | context | text
    path: <source path or null>
    title: <title>
    date: <date>
    hash: <sha256 of source content for integrity>
  generated_by: flow-init
  generated_at: <timestamp>
```

---

## Step 6 — Confirm with User

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<SPEC-ID> Bootstrapped from Source
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Source: <source.path>
Detected:
  • User Stories: <n>
  • Acceptance Criteria: <n>
  • API Endpoints: <n>
  • Data Models: <n>

⚠️  This spec is pre-approved (skipping alignment).
    Original document is embedded in requirement.md#Source

Proceed to generate artifacts?
[yes — generate] / [edit first] / [cancel]
```

**User says "edit":**
- Show editable fields one by one or use `/plan` style

**User says "yes":**
- Write all three files
- Print summary

---

## Step 7 — Finish

```
docs/specs/<SPEC-ID>/
├── requirement.md    ✓ written (with source embedded)
├── test-cases.md     ✓ written
└── spec.yaml         ✓ written (status: approved)

Source provenance preserved in requirement.md#Source

Run /flow:continue to start development.
```

Same worktree question as `/flow:new` (Step 7 and Worktree Setup).

---

## Special Cases

### Source has no ACs
```
⚠️  Source document has no explicit acceptance criteria.
Generating testable ACs from user stories...
```

### Source conflicts with existing spec
```
⚠️  Detected similar spec: SPEC-003 ("User Login")
Continue creating new spec, or update SPEC-003?
[new spec] / [update SPEC-003] / [cancel]
```

### Source is already a spec.yaml
```
⚠️  Source appears to be a spec.yaml file.
Re-importing would create a duplicate.

Options:
[continue] Import as new revision (bump version, keep history)
[validate] Just validate the spec and print state
[cancel]
```

### Multiple source files
If user provides multiple files:
```bash
/flow:init from requirements.md + api.yaml + designs.figma
```

Treat as composite source:
- `requirement.md` ← main requirements.md + any other .md
- `spec.yaml` ← merge api.yaml for technical_contract section
- `provenance.sources[]` ← list all inputs

---

## Integration with Existing Flow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   /flow:init    │────→│  /flow:continue │────→│   /flow:sync    │
│ (bootstrap from │     │  (execute)      │     │  (finish)       │
│  existing doc)  │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │
         │ alternative entry point
         ▼
┌─────────────────┐
│    /flow:new    │
│ (start from     │
│  scratch)       │
└─────────────────┘
```

Both paths converge at `status: approved` → `/flow:continue`.

---

## Examples

### Example 1: From /plan output
```
User: /flow:init
System: Found recent plan in context: "Login Page Redesign"
        Continue with this source? [yes/no]
User: yes
System: SPEC-004 bootstrapped. Run /flow:continue to develop.
```

### Example 2: From file
```
User: /flow:init from docs/plan/oauth-integration.md
System: Reading docs/plan/oauth-integration.md...
        Detected: 3 user stories, 6 ACs, 2 API endpoints
        Generate spec? [yes/edit/cancel]
User: yes
System: SPEC-005 created. Original doc embedded in requirement.md.
```

### Example 3: Quick MVP
```
User: /flow:init "Dark mode toggle in navbar, persists to localStorage"
System: Quick MVP mode — generating minimal spec...
        SPEC-006 created with 2 ACs.
        Run /flow:continue to develop (est: 30 min).
```

---

## Hard Rules

1. **Always preserve source** — requirement.md must have ## Source section
2. **Never overwrite** — if spec_id exists, ask before clobbering
3. **Status is approved** — skip alignment phase entirely
4. **Single agent** — don't launch three-agent debate, just convert
5. **Validate before write** — show summary, get confirmation