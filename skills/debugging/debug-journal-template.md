# Debug Journal — [Bug Title]

**Date:** [YYYY-MM-DD]
**Severity:** [P0-critical | P1-high | P2-medium | P3-low]
**Pattern tag:** [e.g., state-sync, race-condition, null-guard, config-drift, type-mismatch, api-contract, off-by-one, stale-cache]
**Files changed:** [list]

---

## The Bug
[1-2 sentences: what was broken from the user's perspective]

## Root Cause
[The actual mechanism — not the symptom, but WHY it happened at the code level]

## Origin — How It Got There
[Trace back: which commit/PR/decision introduced this? Was it a regression, a design flaw, or an oversight?]
```
Introduced by: [commit hash or "original design"]
When: [date or PR reference]
Context: [what was being built when this was introduced]
```

## How We Missed It
[What test, review, or process gap allowed this to ship? Be specific — "no test coverage for X" is better than "insufficient testing"]

## The Fix
[What changed and why this addresses the root cause, not just the symptom]
```
Fix type: [guard-added | logic-corrected | state-sync-fixed | config-updated | type-narrowed | race-eliminated | other]
Confidence: [pre-skeptic score from Phase 4]
```

## Prevention Rule
[A concrete, actionable rule that would have caught this bug before it shipped. Not "be more careful" — something enforceable.]

**Example prevention rules:**
- "All async state updates must check mounted flag before setState"
- "Every API response handler must validate shape before destructuring"
- "Config values loaded from env must have fallback defaults with type assertions"

## Regression Guard
[What test was added to prevent this from recurring? Include the test name and what it covers.]
```
Test: [test file:test name]
Covers: [what scenario this test guards against]
```

## Related Bugs
[Links to other debug journals with the same pattern tag, if any exist]
