---
name: building
description: Use when plan exists and you're ready to implement. Unified skill merging TDD, execution, subagent dispatch, and inline code review. Follows test-first discipline with fresh subagent per task.
---

# Unified Building

## Iron Law
**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**
Code written before its test must be deleted and rewritten after the test exists.

## Process

### Step 1: Load Plan + Assumption Check
Read the plan file. Create TodoWrite entries for each task. Identify which tasks are independent (can use `parallel-dispatch`).

**Assumption Detector**: Before starting ANY task, list 2-3 assumptions you're making (e.g., "file X exists", "function Y has parameter Z", "library V is installed"). Verify each against the codebase with Grep/Read/Bash. If any assumption is wrong → fix your understanding before writing code. Building on wrong assumptions compounds errors.

### Step 2: For Each Task — TDD Cycle

**RED → GREEN → REFACTOR**:
1. **RED**: Write a test describing expected behavior. Run it → must FAIL. If it passes without code → the test is wrong, fix it.
2. **GREEN**: Write the simplest code that makes the test pass. Run test → must PASS. Fix code (not test) if it fails.
3. **REFACTOR**: Remove duplication, improve names, simplify. Run test → still PASSES. Commit.

### Step 3: Inline Code Review
After each task's TDD cycle, dispatch a reviewer subagent (read `reviewer.md`):

```
Review this diff for: security issues, performance problems,
logic errors, missing edge cases. Be specific. Under 100 words.
```

If reviewer finds issues → fix in current task before moving to next.

**Subagent Verification Rule**: After ANY subagent dispatch (implementer or reviewer), you MUST independently verify their claims. Run the tests yourself. Read the diff yourself. A subagent saying "DONE" is not evidence — only test output you personally observed is evidence.

### Step 4: Task Completion
- Mark TodoWrite task as completed
- Move to next task
- After ALL tasks: run full test suite to catch integration issues

## Subagent Dispatch for Tasks
For complex tasks, dispatch an implementer subagent (read `implementer.md`):
- Provide: task description, relevant file contents (extracted — don't make them read files), test requirements
- Expect: implementation + tests + status (DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED)
- If NEEDS_CONTEXT: provide the specific missing context and re-dispatch once.
- If BLOCKED: subagent output must include `Needs: [exact file/function/info missing]` + `Tried: [what was attempted]`. Never retry without addressing the specific blocker.

## Stopping Points + Checkpoint Rollback
STOP and assess if:
- A task takes 3x longer than estimated
- Tests fail in unexpected ways after 2 fix attempts → **Checkpoint Rollback**: prefer `git stash` (safe — preserves and recovers changes). Use `git checkout .` only if you have no new untracked files — it permanently deletes them with no recovery. Never compound errors by building on broken code.
- You discover the plan has a gap → return to `planning`
- Build/lint errors cascade → invoke `debugging`

## Transition
All tasks done + full test suite passes → invoke `adversarial-testing`, then `verification`.

## Anti-Patterns
- "I'll add tests after" → NO. Test first or delete the code.
- "Already manually tested" → Not evidence. Write the test.
- "This is too simple to test" → Simple code still needs a test.
- Skipping review on "obvious" changes → Review everything.
- Batching task completions → Mark complete immediately.
- Modifying files outside current task scope → Scope creep. Revert.

