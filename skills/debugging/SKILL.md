---
name: debugging
description: Use when encountering any bug, test failure, or unexpected behavior. Enforces root-cause investigation before fixes. 4-phase systematic approach prevents shotgun debugging.
---

# Systematic Debugging

## Iron Law
**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**
If you propose a fix before Phase 2, restart from Phase 1.

## When to Use
- Any test failure or unexpected behavior
- Build errors that aren't obvious typos
- "It works locally but not in CI/production"
- After 2+ failed fix attempts on the same issue

## Phase 1: Observe
1. **Reproduce**: Run the exact command that fails. Copy the full error output.
2. **Read the error**: What does it actually say? Not what you assume.
3. **List assumptions**: What are you assuming about the system state? Verify each.
4. **Isolate**: What's the smallest input that triggers the failure?
5. **Timeline**: When did it last work? What changed since then? (`git log --oneline -10`)

Write down your observations before proceeding. No guessing.

## Phase 2: Hypothesize
Based on Phase 1, form **exactly 2** hypotheses:
- H1: [most likely cause] — because [evidence from Phase 1]
- H2: [alternative cause] — because [evidence from Phase 1]

Each hypothesis must cite specific evidence, not intuition.

## Phase 3: Test
For each hypothesis:
1. Design a test that would **confirm or deny** it
2. Run the test
3. Record the result

If both hypotheses are wrong → return to Phase 1 with new information. Do NOT guess a third hypothesis without more evidence.

## Phase 4: Fix
Only when root cause is confirmed:
1. Write a test that reproduces the bug (RED)
2. Fix the root cause — minimal change (GREEN)
3. Run original failing command → confirm fixed
4. Run full test suite → confirm no regressions

## Escalation
If 3+ fix attempts fail → the bug may be architectural:
- Step back and question assumptions
- Check if the feature design is sound
- Consider if the test itself is wrong
- Ask the user for context you might be missing

## Red Flags — Restart Phase 1
- Proposing a fix without stating root cause
- "Let me try this..." (shotgun debugging)
- Changing multiple things at once
- Fixing symptoms instead of causes
- "It might be..." without evidence

## Anti-Patterns
- Adding try/catch to hide errors instead of fixing them
- Commenting out failing tests
- "Works on my machine" without investigating environment differences
- Blaming the framework without evidence
