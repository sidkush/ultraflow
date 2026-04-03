---
name: receiving-review
description: Use when receiving code review feedback before implementing suggestions. Requires technical verification of each suggestion — bans performative agreement and blind implementation.
---

# Receiving Code Review

## Iron Law
**NEVER AGREE PERFORMATIVELY. VERIFY TECHNICALLY BEFORE IMPLEMENTING.**
Performative agreement: "You're absolutely right!", "Great catch!", "Of course, I should have..." — any praise or concession before verification. Every suggestion gets scrutiny first.

## When to Use
- When a human or subagent provides code review feedback
- Before implementing ANY suggested change from a review
- When review feedback seems unclear or technically questionable

## Process

### Step 1: Read All Feedback
Read the complete review. Don't react to individual comments — understand the full picture first.

### Step 2: For Each Suggestion — Verify

**A. Restate the requirement**: What is the reviewer asking for? State it in your own words.

**B. Check the codebase**: Does the suggestion match reality?
- If they say "function X is missing error handling" → read function X. Is it?
- If they say "this pattern isn't used elsewhere" → grep for it. Is that true?
- If they suggest a library/API → verify it exists and works as described.

**C. Evaluate technical soundness**:
- Does the suggestion actually improve the code?
- Could it introduce new problems?
- Is it within scope of the original task?

### Step 3: Respond with Reasoning

For each suggestion, respond with ONE of (required fields per type):

**AGREE + IMPLEMENT**: Evidence: [file:line confirming the issue]. Change: [what specifically will change]. → Then implement.

**AGREE + DEFER**: Reason: [why out of scope]. Follow-up: [task description to track it].

**PUSHBACK**: Counter-evidence: [specific finding from codebase — file:line or grep result]. Better because: [reason].

**CLARIFY**: Interpretation A: [option]. Interpretation B: [option]. Which do you mean?

### Step 4: YAGNI Check
Before implementing any suggested addition (new feature, new abstraction, new parameter):
- Grep the codebase: is this actually needed NOW?
- **Only applies if codebase has ≥1 existing file to grep against.** On new/empty codebases, skip YAGNI — there's nothing to compare against.
- If not used anywhere yet → pushback. Don't build for hypothetical futures.

## Anti-Patterns
- Implementing every suggestion without checking if they're correct
- Agreeing to avoid social friction
- Adding features the reviewer suggests without verifying they're needed
- Treating review feedback as commands rather than suggestions to evaluate
