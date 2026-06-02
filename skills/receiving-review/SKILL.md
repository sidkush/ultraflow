---
name: receiving-review
description: Use when receiving human, subagent, or PR review feedback before implementing suggestions.
---

# Receiving Code Review

## Iron Law

Never agree performatively. Verify technically before implementing.

Performative agreement includes praise or concession before checking code. Every suggestion gets scrutiny first.

## Process

1. Read all feedback before reacting.
2. Restate each suggestion as a concrete requirement.
3. Verify against HEAD with file reads, grep, tests, or docs.
4. Classify response:
   - `AGREE + IMPLEMENT`: evidence confirms issue; fix now.
   - `AGREE + DEFER`: valid but out of scope; link to deferred item or follow-up.
   - `PUSHBACK`: counter-evidence proves suggestion wrong or harmful.
   - `CLARIFY`: multiple interpretations remain.
5. For additions, run YAGNI check against actual usage.

## AskDB Doctrine Checks

- Trust-layer suggestions require reachability and runtime/invocation-surface evidence.
- Familiar findings require falsified-registry/deferred-bugs consult.
- UI suggestions are correctness issues when labels, states, or errors are unreadable.
- No live LLM/API tests as proof.

## Output

```markdown
Suggestion:
Classification:
Evidence:
Action:
Deferred link:
```

## Anti-Patterns

- Implementing all review comments as commands.
- Socially agreeing before checking.
- Fixing Minor issues while Critical/Important findings remain.
- Accepting a P0/P1 without reachability.
