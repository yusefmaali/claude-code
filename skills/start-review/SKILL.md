---
name: start-review
description: Generates a comprehensive, copy-pasteable prompt to launch a multi-agent code review of the most recently completed implementation phase. Auto-detects the last COMPLETED phase, or accepts an explicit hint (e.g., "/start-review frontend 2" or "/start-review backend 5"). The generated prompt spawns 3+ code-reviewer agents with distinct focus areas and produces a structured review report.
allowed-tools: Read, Write, Glob, Grep
version: 1.0.0
---

## Overview

This skill reads the implementation progress tracker, identifies the most recently completed phase, and generates a comprehensive review prompt that can be copy-pasted into a fresh Claude Code session to launch a multi-agent code review.

## Inputs

- **Arguments (optional):** Stack and/or phase number hints. Examples:
  - `/start-review` — auto-detect the most recently completed phase
  - `/start-review frontend 1` — explicitly target frontend Phase 1
  - `/start-review backend 5` — explicitly target backend Phase 5
  - `/start-review 3` — Phase 3, auto-detect stack from progress tracker
  - `/start-review frontend` — most recently completed frontend phase

## Instructions

### Step 1: Locate Project Files

1. Find the progress tracker: search `docs/plans/` for files matching `implementation-*-progress.md`
2. Find plan files: search `docs/plans/` for files matching `*-plan.md`
3. Find the PRD: `docs/specs/PRD.md`
4. Find the review template: `docs/plans/template-implementation-review.md`
5. Read the progress tracker fully

### Step 2: Determine the Target Phase

Parse the progress tracker to identify the phase to review:

1. **If explicit arguments were provided**, use them directly:
   - If both stack and phase number given → use as-is
   - If only phase number given → scan the progress tracker for that phase number's stack
   - If only stack given → find the most recently COMPLETED phase for that stack

2. **If no arguments**, auto-detect:
   - Scan all phases under `## Backend Implementation Status` and `## Frontend Implementation Status`
   - Find the **last COMPLETED phase** (the highest-numbered phase marked `✅ COMPLETED` that does NOT already have a corresponding review file in `docs/plans/`)
   - If all completed phases already have review files, pick the most recently completed phase (the user may want to re-review)

3. **Check for existing review files:**
   - Search `docs/plans/` for files matching `{stack}-{milestone}-review-phase-{N}*.md`
   - If a review file already exists for this phase, **warn the user** but still generate the prompt (they may want a fresh review)

4. **Extract from the matched phase header:**
   - **Phase number** (e.g., `3`)
   - **Phase title** (e.g., `Progress Indicator — F-PROGRESS, F2.5`)
   - **Task count** (e.g., `7 tasks`)
   - **Stack** (`backend` or `frontend`)

5. **Derive the plan file name** using the pattern: `{stack}-{milestone}-plan.md`
   - The milestone comes from the progress file name: `implementation-{milestone}-progress.md` → milestone = e.g., `p1`

6. **Read the plan file** to extract the full specification for the target phase

### Step 3: Gather Context

Read these files in parallel:
- The **plan file** (`{stack}-{milestone}-plan.md`) — full implementation spec for the target phase
- The **PRD** (`docs/specs/PRD.md`) — product requirements
- The **review template** (`docs/plans/template-implementation-review.md`) — required report structure
- The **progress tracker** (already read) — for completed phases context and deferred issues

From the plan file, extract the **exact section** for the target phase, including:
- All task tables, code snippets, design decisions, implementation notes
- Dependencies on previous phases
- Referenced patterns and conventions

From the progress tracker, extract:
- The target phase's task table (with completion notes)
- Any deferred review issues from previous phases
- Key outcomes from prior phases (for context)

### Step 4: Determine Agent Focus Areas

Design at least 3–4 code-reviewer agents with **non-overlapping focus areas** tailored to the phase content. Agents should cover at least these areas:

1. **Backend/API Alignment** or **Plan Compliance** — verifying the implementation matches the spec
2. **Code Quality & Patterns** — conventions, signals/RxJS patterns, architecture compliance
3. **Test Coverage & Robustness** — test quality, edge cases, error handling
4. **Accessibility & UX** _(only if the phase includes UI components)_ — ARIA, keyboard nav, i18n

Add more at your will.

For each agent, define:
- **Focus area title** (e.g., "Backend DTO Alignment")
- **Files to review** (list the exact new/modified files from the plan's file inventory)
- **Verification checklist** (5–10 specific checks tailored to the phase)
- **Deep analysis questions** (2–3 probing questions requiring thorough code reading)

### Step 5: Generate the Review Prompt

Compose a comprehensive review prompt using this template structure. Use ultrathink and deep think for maximum quality.
This is production code.

````markdown
Conduct a comprehensive multi-agent code review of **{Stack} Phase {N}: {Phase Title}** for the ErgoSafe project.

## Review Context

- **Phase:** {Stack} Phase {N} — {Phase Title} ({task_count} tasks)
- **Branch:** `feature/{milestone}` (latest commit on this branch)
- **Milestone:** {milestone} ({completed_count}/{total_count} tasks completed overall)
- **Prior phases completed:** {list completed phases with 1-line summaries}
- **Deferred issues from previous phases:** {list any deferred issues targeting this phase, or "None"}

## Source of Truth

- `docs/plans/{stack}-{milestone}-plan.md` — Phase {N} section (full implementation spec)
- `docs/plans/implementation-{milestone}-progress.md` — progress tracker with task completion notes
- `docs/specs/PRD.md` — product requirements
- `docs/plans/template-implementation-review.md` — review report structure template

## Phase {N} Specification Summary

{Paste a concise summary of the phase scope: what was supposed to be built, key design decisions, and critical requirements. Include task count, new file count, and test expectations.}

## Agent Assignments

Launch **{N} code-reviewer agents in parallel**, each with a distinct focus area.

### Agent 1: {Focus Area Title}

**Focus:** {1-2 sentence description of what this agent reviews}

**Files to review:**
- `{path/to/file1}` — {purpose}
- `{path/to/file2}` — {purpose}
- ...

**Reference files (read for context, not under review):**
- `docs/plans/{stack}-{milestone}-plan.md` — Phase {N} section
- `docs/plans/implementation-{milestone}-progress.md`
- {any other reference files relevant to this agent}

**Verification checklist:**
- [ ] {Check 1}
- [ ] {Check 2}
- ...

**Deep analysis questions:**
1. {Probing question requiring thorough code reading}
2. {Question about edge cases or correctness}

### Agent 2: {Focus Area Title}
{Same structure as Agent 1}

### Agent 3: {Focus Area Title}
{Same structure as Agent 1}

{### Agent 4+: ... (if applicable)}

---

### Output Format

Each agent should produce:

1. **Verdict**: PASS / FAIL / CONDITIONAL PASS with brief justification
2. **Files Reviewed**: List with line counts
3. **Critical Issues** (blocking): Must be fixed before merge
4. **Important Issues** (non-blocking): Should be fixed soon
5. **Minor Issues** (optional): Style/consistency improvements
6. **Verification Checklist**: Status for each item in their focus area
7. **Deep Analysis Findings**: Answers to the deep questions
8. **Concerns**: Any new issues discovered during review (even if PASS)
9. **Recommendations**: Any suggestions for improvement (even if PASS)
10. **Positive Findings**: Well-implemented patterns worth noting
11. **Code Quality Notes**: Any observations on style, patterns, documentation
12. **PRD Alignment Score**: 0-100% alignment with requirements

Final consolidated report should clearly indicate if the implementation is APPROVED for merge.

---

## Review Instructions

1. Use ultrathink mode, enable deep analysis for comprehensive review
2. Check for edge cases, consider what the implementation might have missed
3. Validate documentation, ensure changelog entries match actual changes
4. Read each file thoroughly - do not skim, understand the implementation fully - before making judgments
5. Compare implementation against PRD and plan specifications
6. Rate each issue as: Critical (blocks release), Important (should fix), or Minor (nice to have)
7. Check for regressions - ensure patterns are followed consistently
8. Be skeptical and thorough - this is production code

---

## Report Output

After all agents complete, synthesize their findings into a single consolidated review report at `docs/plans/{stack}-{milestone}-review-phase-{N}.md`.
Read the template at `docs/plans/template-implementation-review.md` to follow the exact document structure (all 17 sections).

---

#### Remember

I'm a skilled senior software engineer, I'm going to be very skeptical on your work.
Think deeply on it, enable ultrathink mode.
In any subagents let them to think deeply and enable ultrathink mode on them as well.
````

### Step 6: Save and Report

1. **Save the generated prompt** to `docs/plans/{stack}-{milestone}-review-prompt-phase-{N}.md`
   - Example: `docs/plans/frontend-p1-review-prompt-phase-2.md`
   - Overwrite if the file already exists
2. **Report to the user** with a short summary (do NOT print the prompt content on screen):
   - Target: `{Stack} Phase {N}: {Phase Title}`
   - Tasks: `{count} tasks`
   - Agents: `{count} agents` with focus area names
   - Output file: `docs/plans/{stack}-{milestone}-review-prompt-phase-{N}.md`
   - Review report target: `docs/plans/{stack}-{milestone}-review-phase-{N}.md`
   - Existing review: `{Yes/No — warn if overwriting}`

## Key Rules

- **Only write the output prompt file** — do not modify any other project files
- **DO NOT execute the generated prompt** — it's meant for a fresh Claude Code session
- **Include the FULL agent assignments** with exact file paths — the target session needs precise instructions
- **Include file paths from the plan's file inventory** — every new and modified file for this phase
- **Include deferred issues** from previous phases if they target this phase
- **Tailor agent focus areas** to the phase content — don't use generic agents
- **Include the Output Format section VERBATIM** — it is the user's quality standard, do not modify it
- **Include the Review Instructions section VERBATIM** — do not modify it
- **Include the Remember section VERBATIM** — do not modify it
- **Include the Report Output section** with the correct file paths for the review report and template
- **Always include reference to the review template** (`docs/plans/template-implementation-review.md`) in every agent's reference files
- **Be precise about file paths** — the plan specifies exact locations for every file

## File Locations

| Pattern | Purpose |
|---------|---------|
| `docs/plans/implementation-{milestone}-progress.md` | Progress tracker with phase statuses |
| `docs/plans/{stack}-{milestone}-plan.md` | Implementation plan (stack = backend/frontend) |
| `docs/specs/PRD.md` | Product requirements document |
| `docs/plans/template-implementation-review.md` | Review report structure template |
| `docs/plans/{stack}-{milestone}-review-prompt-phase-{N}.md` | Generated review prompt output file |
| `docs/plans/{stack}-{milestone}-review-phase-{N}.md` | Final review report (generated by the review session) |
| `CLAUDE.md` | Project conventions and commands |
