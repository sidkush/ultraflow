---
name: brainstorming
description: Use before creative or implementation work when requirements, operator flow, scope, or value need refinement before planning.
---

# Brainstorming

## Core Principle

Turn vague intent into an operator-readable spec before planning. The output is a scoped design, not code.

## Hard Gate

Do not invoke implementation skills, write code, scaffold, or plan until the design is approved. For complete operator dispatches, run the value check and skip to planning only if the dispatch already contains entry point, happy path, failure paths, acceptance criteria, and out-of-scope.

## Value Check First

Answer before proceeding:

- Improves a real operator/user flow?
- Worth paying engineering cash for?
- Solves active bottleneck, trust gap, support burden, retention risk, or differentiation gap?

If no, defer to `docs/superpowers/triage/deferred-bugs.md`.

## Required Coverage

Every brainstorming pass must cover:

- **Entry point:** where the operator starts.
- **Happy path:** the shortest successful operator flow.
- **Failure modes:** backend dependency failures, empty/error/loading/stale states, and what the operator sees.
- **Failure-state UX:** human-readable error, next action, and recovery path.
- **Acceptance criteria:** binary, observable, phrased in operator-flow language.
- **Out of scope:** explicit boundaries.
- **Operator comprehension:** could a non-author understand every label, state, and error?

## Progressive Flow

1. **Context scan:** read relevant docs/files/recent commits before questions.
2. **Domain scan:** ask at most two questions if the area or scale is unclear.
3. **Option cards:** present exactly three genuinely different approaches: simple, thorough, and creative. Each needs a specific counterfactual.
4. **Deep dive:** ask one question at a time about constraints, edge cases, integration, users, and failure paths.
5. **Spec preview:** present a compact design with entry point, happy path, failure paths, acceptance criteria, and out-of-scope.
6. **Confidence check:** if confidence is below 4/5 after two loops, write a draft with open questions instead of pretending certainty.

## UFSD Write

Write approved specs to `docs/ultraflow/specs/UFSD-YYYY-MM-DD-<feature>.md`.

Include:

```markdown
# UFSD Summary
Feature:
Entry point:
Happy path:
Failure paths:
Scope baseline: In: [...] Out: [...]
Assumptions:
Acceptance criteria:
Confidence:
```

Then invoke `ultraflow:council` if approach choice remains meaningful, otherwise `ultraflow:writing-plans`.

## Anti-Patterns

- Skipping the value check because the request is detailed.
- Asking many questions up front.
- Options that make the same tradeoff.
- Route-exists acceptance criteria instead of operator-entry smoke.
- Failure states hidden behind happy-path conditionals.
- Placeholder labels, UUID dropdowns, `[object Object]`, or state text only the author understands.
