---
name: verification
description: Use before claiming work is complete, fixed, or passing. Requires running verification commands and reading their COMPLETE output before making any success claims. Evidence before assertions.
---

# Verification Before Completion

## Iron Law
**NO SUCCESS CLAIMS WITHOUT FRESH EVIDENCE.**
"I believe it works" is not evidence. A passing test run IS evidence.

## When to Use
- Before saying "done", "fixed", "passing", or "complete"
- Before committing, creating PRs, or marking tasks complete
- After any fix — verify the fix actually works
- Always. Every time. No exceptions.

## Process

### Steps 1–4: Run, Read, Verify
Run the verification command NOW (not earlier, not from a subagent). Read ALL output — not just the last line. Check exit code (0 = success), warnings, and skipped tests.

**If verification FAILS** (exit code ≠ 0, test failures, errors) → invoke `debugging`. Never patch tests to pass — patching is a symptom fix, not a solution.

### Step 5: Quote the Evidence
Required evidence block — all 4 fields:
```
Command: [exact command run]
Exit code: [N]
Result: [outcome line verbatim that proves the claim — e.g., "5 passed, 0 failed" for tests; "Build succeeded in 2.3s" for builds; "0 problems" for linters. Exit code alone is not sufficient — the result must reflect what was verified.]
Full output: [last 10-20 lines verbatim]
```
A summary or partial paste does not satisfy this step.

### Step 6: Scope Creep Guard
Before claiming done, ask: "Did I do ONLY what was asked? Did I add anything extra?" If you added unrequested features, refactoring, or comments → revert the extras. Scope creep is a hallucination of requirements.

### Step 7: Make the Claim
NOW you can say "done" or "fixed" — citing the evidence you just showed.

## Red Flags
- Using words like "should", "probably", "I think", "seems to" → you haven't verified
- Expressing satisfaction before running verification → premature
- Trusting a subagent's report without independent verification → verify yourself
- "It passed last time" → run it again. Things change.

## Anti-Patterns
- Committing without running tests
- Saying "fixed" because the code looks right (it might not be)
- Skipping verification for "trivial" changes
- Reading only the summary line of test output
