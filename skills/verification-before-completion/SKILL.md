---
name: verification-before-completion
description: Use before claiming work is complete, fixed, passing, committed, pushed, or ready for review.
---

# Verification Before Completion

## Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH EVIDENCE
```

Evidence must be from this run, not a subagent report or earlier memory.

## Gate

1. Identify the command or smoke that proves the claim.
2. Run it fresh.
3. Read all output and exit code.
4. Check warnings, skips, and known baseline.
5. Report evidence and limitations.

If verification fails, invoke `ultraflow:systematic-debugging`.

## Required Evidence Block

```markdown
Command:
Exit code:
Result line:
Full output excerpt:
Known baseline comparison:
```

## UI/Browser Smoke Boundary

For UI work, tests are necessary but not sufficient. Operator-flow smoke must start from the real entry point with real login, happy path, one failure mode, clean console, and readable labels/errors.

If the agent did not execute that smoke, write:

`UI smoke: NOT EXECUTED - deferred for manual verification`

Do not claim UI verified unless the smoke actually ran.

## No Live LLM/API Tests

Verification commands must not call live LLM/provider APIs. Mock at the network boundary.

## Scope Guard

Before completion, inspect `git diff` and ask: did this do only what was requested? Revert or explicitly report unrelated extras.

## Banned Completion Language Without Evidence

"should", "probably", "looks good", "seems fixed", "I think", and any success implication before evidence.
