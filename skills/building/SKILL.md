---
name: building
description: Use when plan exists and you're ready to implement. TDD-enforced, UFSD-aware, with invariant preservation and skill chain interrupts.
---

# Unified Building

## Iron Law
**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

## Step 0: UFSD Read + Failure Mode Map
Read the UFSD summary. Extract: scope baseline, invariants, open assumptions, cascade-map.

**Meta-cognitive failure mode map** — before ANY code, identify 5 failure modes for THIS build session:
1. Assumption drift — code works in isolation, fails integrated
2. Test theater — tests pass but don't cover real behavior
3. Scope creep — modifying files outside current task
4. Subagent trust — accepting "DONE" without verification
5. Invariant erosion — each task slightly breaks a contract

Inject top 2 into every implementer/reviewer subagent prompt as "Watch for: [failure mode]".

## Step 1: Load Plan + Assumption Registry
Read plan file. Create TodoWrite entries per task. Flag independent tasks for `parallel-dispatch`.
Before ANY task, log 2-3 assumptions (file exists, function signature, library installed).
Verify each with Grep/Read/Bash. Wrong assumption → fix first. Log to UFSD: `ASSUMPTION: [text] | VALIDATED: [yes/no]`.

## Step 2: TDD Cycle (per task)
**RED → GREEN → REFACTOR**:
1. **RED**: Write test for expected behavior. Run → must FAIL. Passes without code → test is wrong.
2. **GREEN**: Simplest code to pass. Run → must PASS. Fix code (not test) if it fails.
3. **REFACTOR**: Remove duplication, improve names. Run → still PASSES. Commit.

### Invariant Preservation (after GREEN)
Re-verify every invariant from UFSD baseline. Any invariant broken → P0 stop. Diagnose and restore before proceeding.

## Step 3: Inline Review
Dispatch reviewer subagent (read `reviewer.md`): "Review this diff for: security, performance, logic errors, missing edge cases. Under 100 words."
Fix findings before next task. **Verify independently** — run tests yourself, read diff yourself.

## Step 4: Subagent Dispatch (complex tasks)
Dispatch implementer (read `implementer.md`) with: task description, extracted file contents, test requirements.
Statuses: `DONE` (verify) | `DONE_WITH_CONCERNS` (review) | `NEEDS_CONTEXT` (provide + re-dispatch once) | `BLOCKED` (must include `Needs:` + `Tried:`).

## Step 5: Task Completion + UFSD Write
Mark TodoWrite complete immediately. After ALL tasks: run full test suite.
Update UFSD summary: `confidence=[1-5] | scope=[completed|deviated] | invariants=[pass/fail list]`.

## Skill Chain Interrupt
| Condition | Level | Action |
|-----------|-------|--------|
| Task 3x over estimate / tests fail after 2 fix attempts | LOCAL | `git stash`, diagnose, retry |
| Plan gap or spec conflict discovered | UPSTREAM | Return to `planning`, update UFSD |
| Cascading errors or invariant chain broken | HALT | Invoke `debugging`, write `[RESTART REQUESTED]` to UFSD — requires user approval |

Never compound errors. Never auto-RESTART — user must approve.

## Structured Output (per task)
```
Task: [name] | Files: [list] | Tests: [command] → [result]
Invariants: [checked — pass/fail] | Root cause: [yes/no]
```

## Transition
All tasks done + full suite passes → invoke `adversarial-testing`, then `verification`.

## Anti-Patterns
- "I'll add tests after" → Test first or delete.
- "Already manually tested" → Not evidence.
- Skipping review → Review everything.
- Batching completions → Mark complete immediately.
- Files outside task scope → Revert.
