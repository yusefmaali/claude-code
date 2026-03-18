---
name: start-phase
description: Generates a comprehensive, copy-pasteable prompt to start the next implementation phase. Auto-detects the next NOT STARTED phase from the progress tracker, or accepts an explicit phase hint (e.g., "/start-phase frontend 3" or "/start-phase backend 5"). Use when beginning a new implementation phase in a fresh Claude Code session.
allowed-tools: Read, Write, Glob, Grep
version: 1.1.0
---

## Overview

This skill reads the implementation progress tracker, identifies the next phase to implement, and generates a comprehensive prompt that can be copy-pasted into a fresh Claude Code session to begin that phase's implementation.

## Inputs

- **Arguments (optional):** Stack and/or phase number hints. Examples:
  - `/start-phase` — auto-detect the next NOT STARTED phase (scans backend first, then frontend)
  - `/start-phase frontend 3` — explicitly target frontend Phase 3
  - `/start-phase backend 5` — explicitly target backend Phase 5
  - `/start-phase 3` — Phase 3, auto-detect stack from progress tracker
  - `/start-phase frontend` — next NOT STARTED frontend phase

## Instructions

### Step 1: Locate Project Files

1. Find the progress tracker: search `docs/plans/` for files matching `implementation-*-progress.md`
2. Find plan files: search `docs/plans/` for files matching `*-plan.md`
3. Find the PRD: `docs/specs/PRD.md`
4. Read the progress tracker fully

### Step 2: Determine the Target Phase

Parse the progress tracker to identify the next phase to implement:

1. **If explicit arguments were provided**, use them directly:
   - If both stack and phase number given → use as-is
   - If only phase number given → scan the progress tracker for that phase number's stack
   - If only stack given → find the first NOT STARTED phase for that stack

2. **If no arguments**, auto-detect:
   - Scan sections under `## Backend Implementation Status` for the first `### Phase N:` header containing `NOT STARTED`
   - If all backend phases are COMPLETED, scan `## Frontend Implementation Status` similarly
   - If all phases are COMPLETED, inform the user that all phases are done

3. **Extract from the matched phase header:**
   - **Phase number** (e.g., `3`)
   - **Phase title** (e.g., `Progress Indicator — F-PROGRESS, F2.5`)
   - **Task count** (e.g., `7 tasks`)
   - **Stack** (`backend` or `frontend`, from which `## ... Implementation Status` section it falls under)

4. **Derive the plan file name** using the pattern: `{stack}-{milestone}-plan.md`
   - The milestone comes from the progress file name: `implementation-{milestone}-progress.md` → milestone = e.g., `p1`
   - So for frontend P1: `frontend-p1-plan.md`, for backend P1: `backend-p1-plan.md`

5. **Read the plan file** to extract the full specification for the target phase

### Step 3: Gather Context

Read these files in parallel:
- The **plan file** (`{stack}-{milestone}-plan.md`) — full implementation spec
- The **PRD** (`docs/specs/PRD.md`) — product requirements
- The **progress tracker** (already read) — to understand what's been completed so far

From the plan file, identify the **exact section** for the target phase (e.g., `## Phase 3: Progress Indicator`) and extract:
- All subsections (task tables, code snippets, design decisions, implementation notes)
- Dependencies on previous phases
- Referenced patterns and conventions

From the progress tracker, extract:
- All completed phases and their key outcomes (for context)
- Any deferred review issues from previous phases that affect this phase
- The overall progress summary

### Step 4: Generate the Prompt

Compose a comprehensive prompt using this template structure. Use ultrathink and deep think on the task for maximum quality.
This is production code.

```markdown
Implement **{stack} Phase {N}: {Phase Title}** for the ErgoSafe project.

### Context
- The {stack} is at Phase {N} of the {milestone} implementation
- {Summary of what has been completed so far — list completed phases with 1-line summaries}
- {Any deferred issues from previous phases that affect this phase}

### Source of Truth
- `docs/plans/{stack}-{milestone}-plan.md` — Phase {N} section (full implementation spec)
- `docs/plans/implementation-{milestone}-progress.md` — progress tracker
- `docs/specs/PRD.md` — product requirements

### Phase {N} Specification

{Paste the FULL phase section from the plan file here, including:}
- All task tables with task IDs, descriptions, priorities, and notes
- All code snippets and interface definitions
- All design decisions and rationale
- All file paths and locations
- Dependencies on previous phases

### Implementation Guidelines

1. Follow the existing codebase conventions documented in CLAUDE.md
2. {Stack-specific guidelines based on what's in the plan's Architecture Conventions section}
3. After implementing each task, update `docs/plans/implementation-{milestone}-progress.md`:
   - Mark the task as `✅ Completed` in the task table
   - Add implementation notes to the Notes column if the implementation differs from the plan
4. Write tests alongside implementations (TDD where specified in the plan)
5. Ensure all new code compiles and existing tests still pass before moving to the next task

### Task Checklist

{List all tasks from the phase's task table as a checklist:}
- [ ] {Task ID}: {Task description}
- [ ] {Task ID}: {Task description}
- ...

### Verification

After completing all tasks:
1. Run the full test suite (`{test command based on stack}`)
2. Verify all {N} tasks are marked ✅ in the progress tracker
3. Update the progress tracker's changelog section with a summary of what was implemented
```

### Step 5: Save and Report

1. **Save the generated prompt** to `docs/plans/{stack}-{milestone}-prompt-phase-{N}.md`
   - Example: `docs/plans/frontend-p1-prompt-phase-1.md`
   - Overwrite if the file already exists
2. **Report to the user** with a short summary (do NOT print the prompt content on screen):
   - Target: `{Stack} Phase {N}: {Phase Title}`
   - Tasks: `{count} tasks`
   - Output file: `docs/plans/{stack}-{milestone}-prompt-phase-{N}.md`
   - Key dependencies: `{list any previous-phase dependencies}`

## Key Rules

- **Only write the output prompt file** — do not modify any other project files
- **DO NOT execute the generated prompt** — it's meant for a fresh Claude Code session
- **Include the FULL phase specification** in the prompt — the target session won't have access to this conversation's context
- **Include code snippets from the plan** verbatim — they are the implementation spec
- **Include deferred issues** from previous phases if they target this phase
- **Be precise about file paths** — the plan specifies exact locations for every new file
- **Adapt test commands** based on stack: `dotnet test ergosafe.webservice.tests` for backend, `npm test` for frontend

## File Locations

| Pattern | Purpose |
|---------|---------|
| `docs/plans/implementation-{milestone}-progress.md` | Progress tracker with phase statuses |
| `docs/plans/{stack}-{milestone}-plan.md` | Implementation plan (stack = backend/frontend) |
| `docs/specs/PRD.md` | Product requirements document |
| `docs/plans/{stack}-{milestone}-prompt-phase-{N}.md` | Generated prompt output file |
| `CLAUDE.md` | Project conventions and commands |
