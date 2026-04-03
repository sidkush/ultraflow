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

### Step 1: Identify Command
What command proves this works?
- Tests: `npm test`, `pytest`, `cargo test`, etc.
- Build: `npm run build`, `cargo build`, etc.
- Lint: `eslint .`, `ruff check .`, etc.
- Custom: whatever the project uses

### Step 2: Run Fresh
Execute the command NOW. Not "I ran it earlier." Not "the subagent said it passed." NOW.

### Step 3: Read Complete Output
Read the ENTIRE output. Not just the last line. Not just "X tests passed."
- Check for warnings that indicate problems
- Check for skipped tests
- Check exit code (0 = success)

### Step 4: Verify It Confirms Your Claim
The output must specifically confirm what you're about to claim:
- Claiming "tests pass" → output shows "X passed, 0 failed"
- Claiming "build succeeds" → output shows successful build with exit 0
- Claiming "bug is fixed" → the reproduction case no longer fails

### Step 5: Quote the Evidence
Paste the last 10-20 lines of output verbatim in your response. Not a summary — the actual output. This proves to the user (and to yourself) that you really ran it.

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
