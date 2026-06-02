---
name: test-driven-development
description: Use when implementing any feature, bugfix, refactor, or behavior change before writing implementation code.
---

# Test-Driven Development

## Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Violating the letter of TDD is violating the spirit. Code written before RED gets deleted and reimplemented from the test.

## Preflight

Start every executor run with:

- `REASONING EFFORT: <none|low|medium|high|xhigh> - <why>`
- Git branch, status, recent log.
- Test baseline and known floor.
- Ticket/Jira reference if available.
- Scope check: branch in place, no auto-created worktree.

Dirty tree or wrong branch -> halt and ask unless explicitly scoped.

## RED -> GREEN -> REFACTOR

1. **RED:** write the smallest test for one behavior. Run it. It must fail for the expected reason.
2. **GREEN:** write the smallest implementation that passes. No adjacent refactor, no feature extras.
3. **REFACTOR:** clean only after green. Re-run focused tests.
4. **Commit:** commit the focused change with evidence.

GREEN-on-arrival means the test is not proving the new behavior. Improve the test or revise scope before implementation.

## AskDB Additions

- **Trust-layer:** before RED, confirm reachability and runtime assumptions from HEAD. For tenant/auth/audit/path/secrets work, require architect-in-loop and falsified-registry consult.
- **Rule 21:** tests must cover the invocation surface, not only output shape: call sites, subprocess PATH, traversal scope, symlinks, version behavior, thread/process model.
- **No live LLM/API tests:** mock at the network boundary.
- **Operator comprehension:** tests passing does not make unreadable UI states correct.
- **Long-dispatch hygiene:** run focused tests inside folds. Run the full floor once after all folds and after outstanding reviewers/destructors return. Cancel stale long runs when the code changes mid-run.

## Rationalization Table

| Excuse | Reality |
|---|---|
| "Too simple" | Simple changes break. RED is cheap. |
| "Tests after are equivalent" | Tests after verify what you built, not what was required. |
| "Manual smoke is enough" | Manual smoke is not regression proof. |
| "Keep code as reference" | You will adapt it. Delete means delete. |
| "Existing behavior already passes" | Then write the missing behavior or prove the scope is already complete. |

## Red Flags

- Code before RED.
- Test passes immediately.
- Fixing the test to match implementation.
- Mocking the behavior under test.
- Skipping focused verification because a larger suite passed earlier.
- Claiming UI smoke done when it was not run from the real entry point.

If a bug appears during GREEN, invoke `ultraflow:systematic-debugging`.
