---
name: adversarial-testing
description: Use after implementation, before declaring done. Dispatches specialized breaker agents to find bugs, security holes, regressions, and edge cases that normal testing misses.
---

# Adversarial Testing

## Core Principle
If you only test the happy path, you only know the happy path works. Breaker agents actively try to destroy your code from different angles.

## When to Use
- After `building` completes all tasks and tests pass
- Before claiming any non-trivial work is "done"
- After fixing a critical bug (to verify fix is robust)
- When changing security-sensitive code

## Process

### Step 1: Select Breakers
Read `breaker-personas.md`. Choose **3** based on change type:

| Change Type | Recommended Breakers |
|-------------|---------------------|
| API/Backend | Pentester, Chaos Monkey, Regression Hunter |
| Frontend/UI | Chaos Monkey, Compatibility Checker, Regression Hunter |
| Data/Pipeline | Chaos Monkey, Load Tester, Regression Hunter |
| Auth/Security | Pentester, Chaos Monkey, Compatibility Checker |
| Refactor | Regression Hunter, Compatibility Checker, Load Tester |
| New feature | Chaos Monkey, Regression Hunter, Pentester |

### Step 2: Dispatch Breakers
Launch 3 agents in parallel. Each gets:

```
You are [BREAKER_NAME]: [BREAKER_DESCRIPTION].

Target: [WHAT_CHANGED — files, functions, endpoints]
Codebase context: [RELEVANT_CODE — extracted inline]
Existing tests: [TEST_FILE_PATHS]

Your mission: Try to break this code from your specialty angle.

Report format:
## Findings
- [SEVERITY: critical/high/medium/low] [description] — Reproduction: [steps]

## Verdict: BROKEN | FRAGILE | SOLID
- BROKEN: Found critical/high issues that must be fixed
- FRAGILE: Found medium issues worth addressing
- SOLID: Could not break it from my angle

Keep response under 150 words.
```

### Step 3: Triage
Collect all findings. Triage by severity:
- **Critical/High** → MUST fix before proceeding. Return to `building`.
- **Medium** → Fix if quick (<5 min), otherwise document as known limitation.
- **Low** → Document only. Do not fix unless trivial.

### Step 4: Fix-and-Rebreak Cycle
For each critical/high fix:
1. Fix the issue
2. Re-dispatch the SAME breaker that found it
3. Confirm verdict changes to SOLID or FRAGILE
4. If still BROKEN → the fix is incomplete. Try again (max 3 attempts).

### Step 5: Final Verdict
All breakers report SOLID or FRAGILE (with documented medium issues):
→ Proceed to `verification`

Any breaker still reports BROKEN after 3 fix attempts:
→ Escalate to user with findings. Do not silently proceed.

## Token Budget
- Skill load: ~400 tokens
- 3 breaker dispatches: ~900 tokens (300 each)
- Fix-rebreak cycles: ~300 per cycle
- Total: ~1600-2800 tokens depending on findings

## Anti-Patterns
- Skipping adversarial testing for "simple" changes (simple changes break things too)
- Ignoring medium findings without documenting them
- Running all 5 breakers when 3 suffice (wastes tokens)
- Fixing low-severity issues before shipping (perfectionism)
