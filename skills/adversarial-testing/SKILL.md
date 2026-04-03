---
name: adversarial-testing
description: Use after implementation, before declaring done. Dispatches specialized breaker agents to find bugs, security holes, regressions, and edge cases that normal testing misses.
---

# Adversarial Testing

## Core Principle
If you only test the happy path, you only know the happy path works. Breaker agents actively try to destroy your code from different angles.

**When**: After building passes tests. After fixing a critical bug. Before claiming "done" for any non-trivial work.

## Process

### Step 1: Select Breakers
Read `breaker-personas.md`. Choose based on change type and scope:

**Proportional dispatch**: Minor/isolated change (≤2 files, no API surface change) → **1 breaker** (use the first Recommended Breaker from the table row matching your change type). System-wide, API, auth, or security change → **3 breakers**.

| Change Type | Recommended Breakers |
|-------------|---------------------|
| API/Backend | Pentester, Chaos Monkey, Regression Hunter |
| Frontend/UI | Chaos Monkey, Compatibility Checker, Regression Hunter |
| Data/Pipeline | Chaos Monkey, Load Tester, Regression Hunter |
| Auth/Security | Pentester, Chaos Monkey, Compatibility Checker |
| Refactor | Regression Hunter, Compatibility Checker, Load Tester |
| New feature | Chaos Monkey, Regression Hunter, Pentester |

### Step 2: Dispatch Breakers
Launch 1 or 3 agents in parallel (per Step 1). Each gets:

```
You are [BREAKER_NAME]: [BREAKER_DESCRIPTION].

Target: [WHAT_CHANGED — files, functions, endpoints]
Code: [RELEVANT_CODE — extracted inline, max 50 lines]

Return ONLY this block — no prose outside it:
## Findings
- [SEVERITY: critical|high|medium|low] [description]
  Reproduce: [exact input] → expected [X] → got [Y]
  (If no finding at this severity, omit the bullet.)

## Tests attempted (required — min 3 before SOLID):
- [specific test or input you tried]
- [specific test or input you tried]
- [specific test or input you tried]

## Verdict: BROKEN | FRAGILE | SOLID
[MAX 120 words]
```

**Validation**: A SOLID verdict without ≥3 Tests attempted is invalid — treat as FRAGILE.

### Step 3: Triage
Collect all findings. Triage by severity:
- **Critical/High** → MUST fix before proceeding. Return to `building`.
- **Medium** → Fix if quick (<5 min), otherwise document as known limitation.
- **Low** → Document only. Do not fix unless trivial.

### Step 4: Fix-and-Rebreak Cycle
For each critical/high fix:
1. Fix the issue
2. Re-dispatch the SAME breaker — **inject `Delta: [file:lines changed — what was fixed and why]`** into the prompt so it tests the actual fix, not the original issue
3. Confirm verdict changes to SOLID or FRAGILE
4. If still BROKEN → the fix is incomplete. Try again (max 3 attempts).

### Step 5: Final Verdict
All breakers report SOLID or FRAGILE (with documented medium issues):
→ Proceed to `verification`

Any breaker still reports BROKEN after 3 fix attempts:
→ Escalate to user with findings. Do not silently proceed.

**Escalation report format** (required — do not escalate without this):
```
## Adversarial Escalation
Breaker: [name]
Finding: [description of what is broken]
Attempts: 3
  Attempt 1: [what was tried] → still BROKEN because [reason]
  Attempt 2: [what was tried] → still BROKEN because [reason]
  Attempt 3: [what was tried] → still BROKEN because [reason]
Blocked by: [root cause — why all fixes failed]
Needs: [breaker's recommendation for unblocking]
```

## Anti-Patterns
- Skipping adversarial testing for "simple" changes (simple changes break things too)
- Ignoring medium findings without documenting them
- Running all 5 breakers when 3 suffice (wastes tokens)
- Fixing low-severity issues before shipping (perfectionism)
