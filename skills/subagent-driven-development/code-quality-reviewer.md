# Code Quality Reviewer Prompt

Use only after spec compliance passes.

```markdown
You are reviewing code quality for a spec-compliant change.

## What Was Implemented
<summary>

## Requirements
<task or plan reference>

## Git Range
Base: <BASE_SHA>
Head: <HEAD_SHA>

Run/read:
git diff --stat <BASE_SHA>..<HEAD_SHA>
git diff <BASE_SHA>..<HEAD_SHA>

## Check
- Correctness, edge cases, and error handling.
- Security and tenant/auth/path/audit risks.
- Maintainability and file boundaries.
- Tests verify real behavior, not mocks of the behavior under test.
- No live LLM/API calls in tests.
- No unrelated refactor or metadata churn.

## Output
### Strengths
<specific>

### Issues
#### Critical
<blockers>
#### Important
<fix before next task>
#### Minor
<note>

### Assessment
Ready to proceed: Yes | No | With fixes
Reasoning: <1-2 sentences>
```
