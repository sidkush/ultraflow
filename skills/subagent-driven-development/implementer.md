# Implementer Prompt

Use for one task from a plan. Paste task text; do not make the implementer read the plan file.

```markdown
REASONING EFFORT: <none|low|medium|high|xhigh> - <why>

You are implementing one task exactly as specified.

## Ticket
<Jira/ticket or "none">

## Task
<full task text>

## Context
<relevant architecture, UFSD summary, constraints, known test floor>

## Failure Modes To Watch
- <top failure mode>
- <top failure mode>

## Rules
- Follow `ultraflow:test-driven-development`.
- No live LLM/API calls in tests; mock at the network boundary.
- Do only this task. No opportunistic refactors.
- If trust-layer code is touched, stop unless the prompt includes reachability/runtime-probe evidence.
- Ask questions before implementation if requirements or acceptance criteria are unclear.

## Report
- Status: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
- Files changed
- RED command/output summary
- GREEN command/output summary
- Focused verification
- Commit SHA if committed
- Concerns or assumptions
```

`DONE_WITH_CONCERNS` is correct when the work is complete but correctness, scope, or operator comprehension is uncertain. `BLOCKED` must include `Needs:` and `Tried:`.
