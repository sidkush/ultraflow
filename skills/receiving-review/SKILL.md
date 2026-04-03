---
name: receiving-review
description: Use when receiving code review feedback before implementing suggestions. Requires technical verification of each suggestion — bans performative agreement and blind implementation.
---

# Receiving Code Review

## Iron Law
**NEVER AGREE PERFORMATIVELY. VERIFY TECHNICALLY BEFORE IMPLEMENTING.**
"You're absolutely right!" followed by blind implementation is dangerous. Every suggestion gets scrutiny.

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

For each suggestion, respond with ONE of:

**AGREE + IMPLEMENT**: "Valid point. [Evidence from codebase]. Implementing." → Then implement.

**AGREE + DEFER**: "Valid point, but out of scope for this change. Creating a follow-up task."

**PUSHBACK**: "I checked and [evidence]. The current approach is better because [reason]."

**CLARIFY**: "I'm not sure I understand. Do you mean [interpretation A] or [interpretation B]?"

### Step 4: YAGNI Check
Before implementing any suggested addition (new feature, new abstraction, new parameter):
- Grep the codebase: is this actually needed NOW?
- If not used anywhere yet → pushback. Don't build for hypothetical futures.

## Banned Phrases
- "You're absolutely right!"
- "Great catch!"
- "Of course, I should have..."
- Any agreement that comes before verification

## Anti-Patterns
- Implementing every suggestion without checking if they're correct
- Agreeing to avoid social friction
- Adding features the reviewer suggests without verifying they're needed
- Treating review feedback as commands rather than suggestions to evaluate
