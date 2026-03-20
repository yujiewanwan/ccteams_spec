---
name: flow-sync
description: >
  Sync all spec artifacts after human-test passes and prepare the PR.
  Triggers on: "/flow:sync", "sync spec", "ready to merge", "prepare PR",
  "update docs", "done testing". Only run after human-test is complete.
  Do NOT trigger during alignment or agent execution.
---

# flow:sync — Update Artifacts and Prepare PR

## What This Skill Does

Updates all three spec artifacts to reflect what was actually built,
then prints the PR checklist for merge.

---

## Step 1 — Find the Spec

Same scan logic as `/flow:continue`.
Confirm spec is in the correct state:

- `status: done` or `status: human-testing` (all ACs passed) → proceed
- Any other status → warn:
  ```
  ⚠️  spec.yaml status is <status> — sync should only run after human-test passes.
  Run /flow:continue to complete the remaining steps first.
  ```

---

## Step 2 — Update `spec.yaml`

Apply these changes and rewrite the file:

1. `status: done`
2. `updated_at: <today>`
3. Fill in any `test_mapping` entries missing `test_file`
4. Append to `changelog`:
   ```yaml
   - version: "<bumped patch or minor>"
     date: "<today>"
     author: "<author>"
     summary: "<one sentence describing what was built>"
   ```
5. Clear any `failure_note` fields from ACs that are now passing

Print the updated `spec.yaml` in a code block.

---

## Step 3 — Update `requirement.md`

Apply these changes:

1. Add `status: done` at the top
2. Move items from `## Open Questions` to a new `## Decisions` section:
   ```markdown
   ## Decisions
   - [question] → [resolution and date]
   ```
3. Remove the `## Open Questions` section if empty

Print the updated `requirement.md` in a code block.

---

## Step 4 — Update `test-cases.md`

Apply these changes:

1. Add `status: done` and `final_coverage: <actual %>` at the top
2. Add a `Result: PASS` line to each AC section
3. For any AC that had a `failure_note` and was subsequently fixed,
   add a `Fixed: <brief description>` line

Print the updated `test-cases.md` in a code block.

---

## Step 5 — Confirm Before Commit

After updating all three files, ask once:

> Docs updated. Any issues before I commit and create the PR?
> [no, proceed / yes: <describe>]

**User says "no" or "proceed" or "looks good":**
- Continue to Step 6 immediately.

**User describes issues:**
- Fix the relevant doc(s), then ask again.

---

## Step 6 — Commit and Create PR

**Do this automatically — do not ask the user to do it manually.**

### 5a — Commit the updated spec files

```bash
git add docs/specs/<SPEC-ID>/requirement.md
git add docs/specs/<SPEC-ID>/test-cases.md
git add docs/specs/<SPEC-ID>/spec.yaml
git commit -m "docs(<SPEC-ID>): sync spec artifacts — status: done"
```

### 5b — Push current branch

```bash
git push -u origin <current_branch>
```

### 5c — Create PR using `gh`

```bash
gh pr create \
  --title "<PR title from spec>" \
  --body "$(cat <<'EOF'
## <SPEC-ID> — <title>

<requirement.summary>

### Acceptance Criteria
<list AC ids and titles>

### Spec Artifacts
- docs/specs/<SPEC-ID>/requirement.md
- docs/specs/<SPEC-ID>/test-cases.md
- docs/specs/<SPEC-ID>/spec.yaml

🤖 Generated with flow-spec-suite
EOF
)" \
  --base <git.base_branch> \
  --head <current_branch>
```

Print the PR URL after creation.

---

## Step 7 — Final Summary

Print:

```
✓ <SPEC-ID> — <title>
──────────────────────────────────────────────────────
✓ requirement.md     synced
✓ test-cases.md      synced
✓ spec.yaml          synced (status: done)
✓ Committed to branch: <branch>
✓ PR created: <PR URL>
──────────────────────────────────────────────────────
Remaining: Architect reviewer approval before merge.
After merge: run worktree cleanup below.
```

If `git.worktree_enabled: true`, print cleanup commands:

```bash
# Run after PR is merged:
git worktree remove <worktree_path>
git branch -d <branch>
```
