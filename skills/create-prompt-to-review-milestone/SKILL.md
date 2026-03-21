---
name: create-prompt-to-review-milestone
description: Generates a comprehensive, copy-pasteable prompt to launch a multi-agent milestone-level code review of an entire completed milestone (all phases). Supports 3 review modes — backend, frontend, or fullstack (integration). Auto-detects milestone from progress tracker, or accepts an explicit hint (e.g., "/create-prompt-to-review-milestone backend p1", "/create-prompt-to-review-milestone fullstack"). The generated prompt spawns 5+ code-reviewer agents with cross-cutting focus areas.
allowed-tools: Read, Write, Glob, Grep, Bash, Agent
version: 1.0.0
---

## Overview

This skill reads the implementation progress tracker, verifies all phases in a milestone are completed, then generates a comprehensive review prompt for a **milestone-level** code review. Unlike phase-level reviews (which verify "was this phase coded correctly?"), milestone reviews verify:

- **Cross-cutting consistency** — patterns and conventions across all phases
- **Architectural integrity** — Clean Architecture boundaries, DI completeness, regression safety
- **Feature completeness** — PRD compliance across the full feature set
- **Integration correctness** _(fullstack mode)_ — DTO alignment, enum serialization, auth flows, E2E feature traces

The generated prompt is designed to be copy-pasted into a fresh Claude Code session.

## Inputs

- **Arguments (required):** Review mode and optional milestone hint.
  - `/create-prompt-to-review-milestone backend` — backend milestone review, auto-detect milestone
  - `/create-prompt-to-review-milestone frontend` — frontend milestone review, auto-detect milestone
  - `/create-prompt-to-review-milestone fullstack` — full-stack integration review, auto-detect milestone
  - `/create-prompt-to-review-milestone backend p1` — explicitly target backend P1 milestone
  - `/create-prompt-to-review-milestone fullstack p1` — explicitly target full-stack P1 milestone

The first argument MUST be the review mode: `backend`, `frontend`, or `fullstack`.
The second argument is an optional milestone identifier (e.g., `p1`, `mvp`). If omitted, auto-detect from progress tracker.

## Instructions

### Step 1: Parse Arguments and Locate Project Files

1. **Parse arguments** to determine review mode (`backend`, `frontend`, or `fullstack`) and optional milestone.
   - If no review mode provided, ask the user which mode they want.

2. **Find project files:**
   - Progress tracker: search `docs/plans/` for files matching `implementation-*-progress.md`
   - Plan files: search `docs/plans/` for files matching `*-plan.md`
   - PRD: `docs/specs/PRD.md`
   - Spec files: `docs/specs/*.md` (detailed specifications)
   - Review template: `docs/plans/template-implementation-review.md`
   - Existing phase-level review reports: `docs/plans/{stack}-{milestone}-review-phase-*.md`
   - `CLAUDE.md` — project conventions

3. **Read the progress tracker fully** to understand milestone completion status.

### Step 2: Validate Milestone Completion

1. **If milestone specified**, use it. Otherwise, identify the milestone from the progress tracker filename (`implementation-{milestone}-progress.md`).

2. **Verify ALL phases are completed** for the target stack(s):
   - For `backend` mode: all phases under `## Backend Implementation Status` must be `✅ COMPLETED`
   - For `frontend` mode: all phases under `## Frontend Implementation Status` must be `✅ COMPLETED`
   - For `fullstack` mode: ALL phases (both backend and frontend) must be `✅ COMPLETED`
   - If any phase is NOT completed, **warn the user** and ask whether to proceed with a partial milestone review

3. **Extract milestone metadata:**
   - Total task counts (per stack and overall)
   - Total test counts
   - Phase count and phase summaries
   - All deferred review issues from all phases
   - Feature list (from plan overview section)

4. **Check for existing milestone review:**
   - Search for `docs/plans/{stack}-{milestone}-review-milestone.md` (or `fullstack-{milestone}-review-milestone.md`)
   - If exists, warn the user but proceed

### Step 3: Gather Context

Read these files in parallel:

**For `backend` mode:**
- `docs/plans/backend-{milestone}-plan.md` — full backend implementation spec
- `docs/specs/PRD.md` — product requirements
- `docs/plans/template-implementation-review.md` — review report structure
- All existing phase-level review reports for reference

**For `frontend` mode:**
- `docs/plans/frontend-{milestone}-plan.md` — full frontend implementation spec
- `docs/specs/PRD.md`
- `docs/plans/template-implementation-review.md`
- All existing phase-level review reports for reference

**For `fullstack` mode:**
- `docs/plans/backend-{milestone}-plan.md` AND `docs/plans/frontend-{milestone}-plan.md`
- `docs/specs/PRD.md` AND all detailed spec files (`docs/specs/0*.md`)
- `docs/plans/template-implementation-review.md`
- All existing phase-level review reports for reference

From the progress tracker, extract:
- All phase summaries with task counts
- All deferred review issues with severity and target
- Changelog highlights (for key implementation decisions)

### Step 4: Explore the Codebase for File Inventory

**This is critical.** Unlike phase-level reviews where the plan lists exact files, a milestone review spans the entire feature set. You MUST explore the actual codebase to build accurate file inventories.

Use the `Agent` tool with `subagent_type=Explore` to discover all relevant files. The explore agent should:

**For `backend` mode**, explore:
- `src/webservice/ergosafe.webservice.domain/` — entities, enums, interfaces, DTOs, exceptions, configurations
- `src/webservice/ergosafe.webservice.services/` — business logic, email, authorization
- `src/webservice/ergosafe.webservice.api/` — controllers, validators, middleware, authorization, workers, DI
- `src/webservice/ergosafe.webservice.data/` — DbContext, configurations
- `src/webservice/ergosafe.webservice.migrations/` — migration files
- `src/webservice/ergosafe.webservice.tests/` — all test files (unit + integration)

**For `frontend` mode**, explore:
- `src/frontend/src/app/modules/assessment/models/` — enums, DTOs, interfaces
- `src/frontend/src/app/modules/assessment/services/` — HTTP services, state services
- `src/frontend/src/app/modules/assessment/components/` — all components (TS + HTML)
- `src/frontend/src/app/modules/assessment/guards/` — route guards
- `src/frontend/src/app/modules/assessment/utils/` — utilities
- `src/frontend/src/app/modules/auth/interceptors/` — error interceptor
- `src/frontend/src/app/modules/shared/` — shared components and services
- `src/frontend/public/i18n/` — translation files
- All `*.spec.ts` test files

**For `fullstack` mode**, explore both backend AND frontend directories above, plus:
- `src/frontend/src/environments/` — environment config
- Backend JSON configuration files (`appsettings*.json`)
- Backend seed data service

For each file found, collect the **full relative path** and **line count** (`wc -l` via Bash).

Organize files into categorized tables (models, services, controllers, components, tests, etc.) in the generated prompt.

### Step 5: Design Agent Focus Areas

Design **at least 5 code-reviewer agents** with **non-overlapping, cross-cutting focus areas**. Agents must NOT be phase-by-phase — they examine the entire milestone through a specific lens.

#### Backend Mode Agents (5-6 agents)

| Agent | Focus Area | Scope |
|-------|-----------|-------|
| 1 | **PRD & Plan Compliance** | Verify every feature is fully implemented per spec. Cross-reference plan tasks against actual code. |
| 2 | **Domain Model & Data Layer** | Entities, enums, DbContext, migration, indexes, naming conventions, Clean Architecture boundaries. |
| 3 | **Business Logic & Services** | All service implementations: business rules, edge cases, error handling, idempotency, transactional boundaries. |
| 4 | **API Layer, Security & Middleware** | Controllers, DTOs, validators, middleware, authorization policies, OWASP vulnerabilities, endpoint routing. |
| 5 | **Test Quality & Coverage** | All test files: coverage, assertion quality, test isolation, mock correctness, integration test architecture. |
| 6 | **Cross-Cutting Architectural Integrity** | DI completeness, naming consistency, exception patterns, regression safety, deferred issue re-evaluation. |

#### Frontend Mode Agents (5 agents)

| Agent | Focus Area | Scope |
|-------|-----------|-------|
| 1 | **PRD & Plan Compliance** | Feature completeness, plan task coverage, requirement gaps. |
| 2 | **Component Architecture & Framework Patterns** | Standalone components, OnPush, signals, lifecycle, Material integration, routing, guards. |
| 3 | **State Management, Services & Error Handling** | Signal-based state, HTTP services, caching, logout cleanup, cross-service interactions, interceptors. |
| 4 | **Accessibility & i18n** | ARIA attributes, keyboard navigation, transloco completeness, no hardcoded strings, i18n key organization. |
| 5 | **Test Quality & Coverage** | All spec files: isolation, mock quality, assertion correctness, TranslocoTestingModule consistency, coverage gaps. |

#### Fullstack Mode Agents (5-6 agents)

These focus on **integration concerns that individual stack reviews cannot catch**:

| Agent | Focus Area | Scope |
|-------|-----------|-------|
| 1 | **API Contract Alignment** | DTO field names/types/nullability match between C# records and TS interfaces. URL paths, HTTP methods, query params, content types. |
| 2 | **Enum & Type Serialization** | C# enum serialization format (int vs string) matches TS enum expectations. Threshold values consistent. Role systems not conflated. |
| 3 | **Authentication & Authorization Integration** | JWT flow, interceptor ordering, 403 handling split, ErgoRole hierarchy, policy-controller mapping, service-layer second-gate. |
| 4 | **End-to-End Feature Flows** | Complete user journeys: assessment → discomfort → medical case → supervisor dashboard. Feature flow completeness per PRD. |
| 5 | **i18n & Data Seeding Integration** | Seed data keys in it.json, namespace structure, transloco references complete, recommendation key resolution. |
| 6 | **Test Coverage & Integration Test Gaps** | Cross-boundary test scenarios, InMemory EF limitations, mock fidelity, enum serialization round-trips, contract drift detection. |

**Adapt these tables** based on the actual codebase content — add or remove agents as needed. The key constraint is: agents must be **cross-cutting** (not phase-specific) and have **non-overlapping** file assignments.

For each agent, define:
- **Focus area title** (e.g., "API Contract Alignment")
- **1-2 sentence description** of what this agent reviews
- **Files to review** — exact paths from the codebase exploration, with line counts
- **Reference files** — plans, specs, PRD, review template (read for context, not under review)
- **Verification checklist** (8–14 specific checks tailored to the focus area)
- **Deep analysis questions** (3–5 probing questions requiring thorough code reading)

### Step 6: Generate the Review Prompt

Compose a comprehensive review prompt. Use ultrathink and very deep think for maximum quality. This is production code.

The prompt must include these sections in order:

1. **Title** — `# {Stack} {Milestone} Milestone Review — Comprehensive Code Review Prompt` (for fullstack: `Full-Stack {Milestone} Milestone Review — Unified Integration Review`)
2. **Review Context** — milestone, branch, total tasks, features, test counts, deferred issues, prior review count
3. **Source of Truth** — file paths for plans, specs, PRD, review template, phase-level review reports
4. **Key Differences from Phase-Level Reviews** — explain what milestone review checks that phase reviews cannot
5. **Complete File Inventory** — categorized tables of all files with paths and line counts
6. **Agent Assignments** — each agent with full structure (focus, files, reference files, checklist, deep questions)
7. **Output Format** section — **VERBATIM** (see below)
8. **Review Instructions** section — **VERBATIM** (see below)
9. **Report Output** section — target file path and template reference
10. **Remember** section — **VERBATIM** (see below)

#### Verbatim Sections

These MUST be included exactly as written:

**Output Format:**
````
## Output Format

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
````

**Review Instructions:**
````
## Review Instructions

1. Use ultrathink mode, enable deep analysis for comprehensive review
2. Check for edge cases, consider what the implementation might have missed
3. Validate documentation, ensure changelog entries match actual changes
4. Read each file thoroughly - do not skim, understand the implementation fully - before making judgments
5. Compare implementation against PRD and plan specifications
6. Rate each issue as: Critical (blocks release), Important (should fix), or Minor (nice to have)
7. Check for regressions - ensure patterns are followed consistently
8. Be skeptical and thorough - this is production code
````

**Remember:**
````
## Remember

I'm a skilled senior software engineer, I'm going to be very skeptical on your work.
Think deeply on it, enable ultrathink mode.
In any subagents let them to think deeply and enable ultrathink mode on them as well.
````

### Step 7: Save and Report

1. **Save the generated prompt** to `docs/plans/{stack}-{milestone}-prompt-milestone-review.md`
   - Backend example: `docs/plans/backend-p1-prompt-milestone-review.md`
   - Frontend example: `docs/plans/frontend-p1-prompt-milestone-review.md`
   - Fullstack example: `docs/plans/fullstack-p1-prompt-milestone-review.md`
   - Overwrite if the file already exists

2. **Report to the user** with a short summary (do NOT print the prompt content on screen):
   - Review mode: `{backend / frontend / fullstack}`
   - Milestone: `{milestone}`
   - Tasks: `{count} tasks across {N} phases`
   - Agents: `{count} agents` with focus area names listed
   - Files inventoried: `{count} primary files (~{N} lines)`
   - Output file: `docs/plans/{stack}-{milestone}-prompt-milestone-review.md`
   - Review report target: `docs/plans/{stack}-{milestone}-review-milestone.md`
   - Existing milestone review: `{Yes/No — warn if overwriting}`
   - Deferred issues flagged: `{count} issues from prior phase reviews`

## Key Rules

- **Only write the output prompt file** — do not modify any other project files
- **DO NOT execute the generated prompt** — it's meant for a fresh Claude Code session
- **Explore the actual codebase** — do not rely solely on plan file inventories for file paths
- **Include exact file paths with line counts** — the target session needs precise instructions
- **Cross-cutting agents only** — agents must NOT be organized by phase
- **For fullstack mode**, agents MUST span both backend and frontend files
- **Include ALL deferred issues** from all phase-level reviews for re-evaluation
- **Include the Output Format section VERBATIM** — do not modify it
- **Include the Review Instructions section VERBATIM** — do not modify it
- **Include the Remember section VERBATIM** — do not modify it
- **Include the Report Output section** with correct file paths for review report and template
- **Always reference the review template** (`docs/plans/template-implementation-review.md`)
- **Include phase-level review reports** as reference files (for Comparison with Previous Reviews section)
- **Fullstack agents focus on INTEGRATION** — not repeating what backend/frontend reviews already cover
- **Be precise about file paths** — explore the codebase, verify paths exist, include line counts

## File Locations

| Pattern | Purpose |
|---------|---------|
| `docs/plans/implementation-{milestone}-progress.md` | Progress tracker with phase statuses |
| `docs/plans/{stack}-{milestone}-plan.md` | Implementation plan (stack = backend/frontend) |
| `docs/specs/PRD.md` | Product requirements document |
| `docs/specs/0*.md` | Detailed specification documents |
| `docs/plans/template-implementation-review.md` | Review report structure template |
| `docs/plans/{stack}-{milestone}-review-phase-*.md` | Phase-level review reports (prior context) |
| `docs/plans/{stack}-{milestone}-prompt-milestone-review.md` | Generated milestone review prompt (output) |
| `docs/plans/{stack}-{milestone}-review-milestone.md` | Final milestone review report (generated by review session) |
| `CLAUDE.md` | Project conventions and commands |
