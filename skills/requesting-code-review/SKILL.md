---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements.
---

# Requesting Code Review

## Core Principle

Review early enough that defects do not cascade. Reviewers receive exact context, not session history.

## When Required

- After each task in `ultraflow:subagent-driven-development`.
- After a major feature/fold.
- Before merge or push when the diff is non-trivial.
- When trust-layer code changes.

## Input Template

Collect:

```bash
git rev-parse <base>
git rev-parse HEAD
git diff --stat <base>..HEAD
```

Dispatch with:

```markdown
BASE_SHA: <sha>
HEAD_SHA: <sha>
WHAT: <what changed>
PLAN: <plan/task/spec link or pasted text>
RISK CLASS: Feature/UI | Trust-layer
```

Use `code-reviewer.md` as the reviewer prompt.

## Severity

| Severity | Meaning | Action |
|---|---|---|
| Critical | Blocks merge; security/data loss/broken core flow | Fix now |
| Important | Should fix before proceeding; requirement gap, edge case, maintainability risk | Fix before next task unless explicitly deferred |
| Minor | Note or defer | Track if useful |

Push back when the reviewer is technically wrong, but cite code or test evidence. Do not performatively agree.

## Two-Stage Integration

For subagent execution, this skill complements the mandatory sequence:

1. Spec compliance review.
2. Code quality review.

Do not use a general code review as a substitute for spec compliance.
