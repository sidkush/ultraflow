---
name: planning
description: Use when spec/requirements exist, before touching code. Generates implementation plans with precise task granularity, file paths, test commands, and git steps. Self-reviews via plan-reviewer subagent.
---

# Streamlined Planning

## Core Principle
A plan is a contract. Every step must be precise enough that a subagent can execute it without asking questions. No "TBD", no "similar to above", no pseudocode.

## When to Use
- After discovery produces an approved spec
- After council selects an approach
- Before ANY implementation begins
- Skip only for single-file, single-function changes where the change is obvious

## Plan Structure

Write plan to `docs/ultraflow/plans/YYYY-MM-DD-<feature>.md`:

```markdown
# Plan: [Feature Name]
**Spec**: [link to spec file]
**Approach**: [chosen council approach or direct description]
**Branch**: `feature/<name>`

## Tasks
### Task 1: [Name] (~N minutes)
- **Files**: `path/to/file.ext` (create|modify)
- **Intent**: [What this code does — NOT full code blocks]
- **Test**: `[exact command]` → expects [specific output]
- **Commit**: `[commit message]`

### Task 2: ...

## Fingerprint
[1-line summary of what this plan produces when complete]
```

## Rules

1. **2-5 minute tasks**: Each task should take a subagent 2-5 minutes. Split larger tasks.
2. **Code intent for simple sections**: Describe WHAT the code does, not the full implementation. Saves ~50% tokens.
3. **Code blocks for critical sections**: Auth logic, data models, API contracts, security checks, and migration schemas MUST include actual code blocks to prevent implementation divergence across subagents.
4. **Exact file paths**: Every task must name exact files to create or modify.
4. **Test commands with expected output**: Not just "run tests" but the exact command and what success looks like.
5. **Task independence**: Mark dependencies explicitly. Independent tasks can run in parallel.
6. **Commit per task**: Each task gets its own commit message.
7. **Fingerprint**: Every plan ends with a 1-line fingerprint summarizing the expected end state. When the plan is referenced later, compare current state against fingerprint to detect staleness.

## Self-Review

After writing the plan, dispatch the `plan-reviewer` subagent (read `plan-reviewer.md` in this directory for the prompt). The reviewer checks:

- [ ] Every spec requirement is covered by at least one task
- [ ] No task says "TBD", "TODO", "similar to", or references another task's code
- [ ] All file paths exist or are marked as "create"
- [ ] Test commands are executable (not pseudocode)
- [ ] Task order respects dependencies
- [ ] Names are consistent (same function/variable name throughout)

If review fails → fix and re-review. Do NOT proceed to building with review failures.

## Transition
When plan passes review → invoke `building` with plan path.
If plan reveals need for parallel work → note which tasks are independent for `parallel-dispatch`.

## Anti-Patterns
- Writing full code blocks in plans (wastes tokens, gets stale)
- Tasks longer than 10 minutes (too coarse for subagents)
- "Test it works" without specifying HOW
- Skipping self-review ("the plan looks fine")
