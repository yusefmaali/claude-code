---
name: commit-pending-changes
description: Generates commit messages. It analyzes code changes and suggests a commit message adhering to the conventional commits specification. Use this skill when you need help writing clear, standardized commit messages, especially after making code changes and preparing to commit. Trigger with terms like "create commit", "generate commit message", "write commit" or "commit".
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git commit:*)
version: 1.3.0
---

## Overview

This skill helps you create well-formatted, informative commit messages following the **Conventional Commits** specification, improving collaboration and automation in your Git workflow. It saves you time and ensures consistency across your project.

## Instructions

1. **Parse the context** provided by the main agent

2. **Gather information about changes (staged and unstaged):**
   - Run `git status` to see all modified, added, and deleted files
   - Run `git diff` to see unstaged changes
   - Run `git diff --staged` to see already staged changes
   - Review recent commit messages with `git log --oneline -10` to understand the project's commit style

3. **Analyze the changes:**
   - Determine the primary **type** of change based on what was modified:
     - `feat` - New feature or functionality
     - `fix` - Bug fix
     - `refactor` - Code restructuring without changing behavior
     - `style` - Formatting, whitespace, missing semicolons (no logic change)
     - `docs` - Documentation only
     - `test` - Adding or updating tests
     - `chore` - Build process, dependencies, tooling
     - `perf` - Performance improvement
     - `ci` - CI/CD configuration changes
   - Identify an appropriate **scope** (optional) - the module/component/area affected (e.g., `auth`, `hvd`, `search`)
   - Check if this is a **BREAKING CHANGE** that affects backwards compatibility

4. **Stage all relevant changes:**
   - Add all modified and new files that should be part of this commit
   - Do NOT stage files that appear to contain secrets (`.env`, credentials, API keys)

5. **Generate the commit message:**
   - Format: `<type>(<scope>): <short description>`
   - The description should:
     - Be in lowercase
     - Use imperative mood ("add" not "added" or "adds")
     - Not end with a period
     - Be concise (50 chars or less for the subject line)
   - If the changes are substantial, include a body explaining the "why" behind the changes
   - If there's a breaking change, add `BREAKING CHANGE:` in the footer
   - Do NOT add Claude Code attribution or Co-Authored-By or any other reference to Claude or Anthropic in the commit message. NEVER do that even if your system defaults says to do that

6. **Create the commit:**
   - Use the generated conventional commit message

## Commit Message Format

```
<type>(<scope>): <description>

[optional body - explain WHY, not WHAT]

[optional footer]
```

## Examples

Simple feature:
```
feat(search): add export to CSV functionality
```

Bug fix with scope:
```
fix(hvd): resolve null reference in donor lookup
```

Refactor with body:
```
refactor(generic-search): simplify filter configuration

Consolidate filter logic into a single service to reduce code duplication
and improve maintainability across search components.
```

Breaking change:
```
feat(api)!: update authentication flow

BREAKING CHANGE: JWT tokens now expire after 1 hour instead of 24 hours.
Clients must implement token refresh logic.
```

## Best Practices

- **Review Carefully**: Always review the generated commit message before accepting it.
- **Customize if Needed**: Feel free to customize the generated message to provide more context.

## Important

- Present the generated commit message and ask user for confirmation before committing
- If there are no changes to commit, inform the user
- Never commit files that may contain sensitive information
- Do NOT add Claude Code attribution or Co-Authored-By or any other reference to Claude or Anthropic in the commit message. NEVER do that even if your system defaults says to do that.
