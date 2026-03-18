---
name: update-review-docs
description: Updates implementation review documents, progress tracker, and changelog after code review fixes are applied. Accepts an optional file name hint as argument (e.g., "/update-review-docs phase-8-post-fixes" or "/update-review-docs backend-p1-review-phase-8"). Use after review issues have been resolved and committed.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git log:*), Bash(git diff:*), Bash(git status:*), Bash(dotnet test:*), Bash(dotnet build:*)
version: 1.0.0
---

## Overview

This skill updates implementation review documents and the progress tracker after code review issues have been resolved. It ensures documentation stays in sync with the codebase.

## Inputs

- **Argument (optional):** File name hint for the review document to update. Examples:
  - `phase-8-post-fixes` — matches `*phase-8-post-fixes*`
  - `backend-p1-review-phase-8` — exact prefix match
  - If omitted, search `docs/plans/` for the most recently modified `*-review-*` file

## Instructions

### Step 1: Identify the Review Document

1. If an argument was provided, use it to find the review file:
   - Search `docs/plans/` for files matching `*{argument}*`
   - If no match, try `docs/plans/{argument}.md`
   - If still no match, inform the user
2. If no argument, find the most recently modified `*review*` file in `docs/plans/`
3. Read the review document fully

### Step 2: Gather Current State

Run these in parallel:
- `git log --oneline -15` to understand recent commits since the review
- `git status` to check for uncommitted changes (warn if any)
- Read the review document template at `docs/plans/template-implementation-review.md` for structural reference
- Read `docs/plans/implementation-p1-progress.md` (or the appropriate milestone progress file)
- Read `docs/plans/implementation-p1-changelog.md` (or the appropriate milestone changelog file)

### Step 3: Analyze What Changed

From the review document, identify:
- **Original issues** and their resolution status
- **Post-review issues** (discovered during post-fix review) and their status
- **Deferred issues** that remain open
- The **commits** that resolved issues (from git log)
- The **current HEAD** commit hash

Cross-reference with code to verify:
- Which issues are truly resolved (code changes match resolutions described)
- Whether new deferred issues exist that aren't documented
- Test counts match (check last test run output or run `dotnet test` / `npm test` if uncertain)

**DO NOT recalculate line numbers in files-reviewed tables** — they are expensive to compute and not valuable for documentation purposes. Keep existing line counts as-is.

### Step 4: Update the Review Document

Update these sections carefully (validate the entire document structure against the template):

1. **Metadata Block:**
   - Update `HEAD` commit reference to current HEAD
   - Update `Last Updated` with today's date and a summary of what changed

2. **Executive Summary:**
   - Update test counts if changed
   - Update issue counts (resolved vs deferred)
   - Update PRD alignment if recalculated

3. **Build & Test Status:**
   - Update test pass count
   - Ensure it reflects the current state (post all fixes)

4. **Issue Statuses:**
   - Mark resolved issues as `✅ Resolved` with description of what was done
   - Mark deferred issues as `⏳ Deferred` with target and justification
   - Add resolution descriptions for newly resolved issues

5. **Verification Checklists:**
   - Update check statuses (⚠️ → ✅ for resolved items)
   - Add notes describing the resolution

6. **Action Items:**
   - Update status column for resolved items
   - Keep deferred items with their target milestone

7. **Changelog:**
   - Append new entries at the bottom with today's date
   - One entry per fix commit or fix batch

8. **Conclusion / Next Steps:**
   - Update test counts, issue counts
   - Reflect current state accurately

### Step 5: Update the Progress Tracker

Read and update `docs/plans/implementation-<milestone>-progress.md`:

1. **Phase task descriptions:** If fixes changed the architecture or moved responsibilities between services, update the relevant task `Notes` column to reflect reality.

2. **Deferred Review Issues subsection:** If the review has deferred (unresolved) issues:
   - Check if a `#### Deferred Review Issues (Phase N)` subsection exists after the phase task table
   - If it doesn't exist and there ARE deferred issues, create it:
     ```markdown
     #### Deferred Review Issues (Phase N)

     | # | Issue | Severity | Target | Status | Review Doc |
     |:-:|-------|----------|--------|:------:|------------|
     | P{phase}-#{n} | {description} | {severity} | {target} | ⏳ {status} | [{review name}]({review-file}.md) |
     ```
   - If it exists, update statuses (⏳ Open → ✅ Resolved) or add new deferred issues
   - Convention: issue IDs are `P{phase}-#{issue_number}` (e.g., `P8-#1`)

3. **Changelog section:**
   - Add a new entry at the **top** of the current date section (or create a new date header if needed)
   - Entry format: `- **{Title}**: {description of what was done}`
   - Keep only the **10 most recent entries** in the progress file
   - If adding exceeds 10, move the oldest entry to `implementation-<milestone>-changelog.md`
   - Always update the `Last Updated` date in the document header
   - Convert relative dates to absolute dates (e.g., "Thursday" → "2026-03-05")

### Step 6: Validate and Report

Before finishing:
- Verify the review document structure matches the template (all 17 sections present where applicable)
- Verify no ⏳ deferred issues are missing from the progress tracker
- Verify no ✅ resolved issues still show as ⏳ in the progress tracker
- Verify test counts are consistent across all documents

Report to the user:
- Which files were updated
- Summary of changes (issues resolved, deferred, new entries added)
- Any inconsistencies found

## Key Rules

- **DO NOT recalculate line numbers** in Files Reviewed tables — too expensive, not useful
- **DO NOT run full test suites** unless the user explicitly requests verification or test counts are uncertain
- **DO NOT modify code files** — this skill only updates documentation
- **Preserve existing content** — only update statuses, counts, and add new entries
- **Be careful with changelog rotation** — always verify entries are in the archive before removing from progress
- **Use absolute dates** in all documentation (never relative like "yesterday" or "last week")
- **Match the existing document tone** — direct, factual, no filler

## File Locations

| File | Purpose |
|------|---------|
| `docs/plans/template-implementation-review.md` | Review document structure template |
| `docs/plans/implementation-{milestone}-progress.md` | Progress tracker with task tables and changelog |
| `docs/plans/implementation-{milestone}-changelog.md` | Changelog archive (overflow from progress tracker) |
| `docs/plans/{stack}-{milestone}-review-phase-{n}[-post-fixes].md` | Review documents |

## Examples

```bash
# Update the Phase 8 post-fix review docs
/update-review-docs phase-8-post-fixes

# Update using exact file name
/update-review-docs backend-p1-review-phase-8-post-fixes

# Auto-detect the most recently modified review
/update-review-docs
```
