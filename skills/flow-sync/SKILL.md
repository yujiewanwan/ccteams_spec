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

## Step 5 — Worktree Cleanup (if applicable)

If `git.worktree_enabled: true`, print:

```bash
# After PR is merged, clean up the worktree:
git worktree remove <worktree_path>
git branch -d <branch>
```

---

## Step 6 — PR Checklist

Print the final checklist:

```
PR checklist — SPEC-001
──────────────────────────────────────────────────────
[ ] docs/specs/SPEC-001/requirement.md   status: done, decisions filled
[ ] docs/specs/SPEC-001/test-cases.md    status: done, results filled
[ ] docs/specs/SPEC-001/spec.yaml        status: done, changelog updated
[ ] All three files committed in the same PR as the code
[ ] PR diff shows both spec changes and code changes
[ ] Architect reviewer checklist complete before merge
──────────────────────────────────────────────────────
After merge: these three files on <base_branch> are the
baseline for the next iteration of this feature.
```
