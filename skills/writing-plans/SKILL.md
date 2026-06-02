---
name: writing-plans
description: Use when a spec or requirements exist for a multi-step task, before touching implementation files.
---

# Writing Plans

## Overview

Write implementation plans that a competent executor can run without session memory. Plans must be evidence-bound, operator-flow oriented, and decomposed by rollback value.

## Pre-Plan Gates

1. **Value check:** if the work does not improve a real flow or reduce a real trust/support/retention/differentiation gap, defer it.
2. **Preflight GO/NO-GO:** record branch, git status, recent log, and test baseline. Dirty tree halts unless scoped by the operator.
3. **UFSD read:** read the active file under `docs/ultraflow/specs/`.
4. **Runtime probe before lock:** verify live call paths, call frequency, schema availability, library/runtime semantics, deployment topology, and invocation surface (`<name>(`, PATH, traversal, symlink, version, thread/process assumptions).
5. **Risk classification:** mark trust-layer vs feature/UI. Trust-layer requires architect probe, reachability, falsified-registry consult, generic destructor dispatch, and explicit review per commit.

## Plan Must Include

- Entry point from the operator's real app/workflow.
- Happy path.
- Failure path per backend or external dependency.
- Smoke from the operator entry, not a direct URL or isolated route.
- Binary observable acceptance criteria.
- Rollback/risk notes.
- Operator-comprehension check.
- Commit tier T1/T2/T3 and why.
- Jira/ticket binding if available.

## Plan Must Exclude

- Line-count predictions.
- Test-delta predictions.
- File-by-file staging instructions.
- Calendar estimates.
- Vague placeholders: TBD, TODO, "etc.", "handle edge cases", "similar to".

## Decomposition

Default is one plan = one commit. Split only when blast radius differs, rollback boundaries matter, trust-layer risk must be isolated, lifecycle correctness is independently testable, or staged rollout aids operator validation.

## Plan Shape

Save to `docs/ultraflow/plans/YYYY-MM-DD-<feature>.md`.

```markdown
# [Feature] Implementation Plan

**Reasoning effort:** <none|low|medium|high|xhigh> - <why>
**Ticket:** <id or none>
**UFSD:** <docs/ultraflow/specs/...>
**Commit tier:** T1|T2|T3 - <why>
**Risk class:** Feature/UI | Trust-layer
**Goal:** <operator-flow outcome>

## Entry Point
<real operator start>

## Happy Path
<operator sequence>

## Failure Paths
- <dependency>: <failure UX/recovery>

## Acceptance Criteria
- [ ] <binary observable criterion>

## Tasks
### Task 1: <name>
Intent:
Files:
RED:
GREEN:
Focused verification:
Rollback:
Commit:

## Operator-Comprehension Check
<labels/errors/states a non-author must understand>

## Counterfactual
Strongest argument against this plan:
Accepted because:
```

## Self-Review

Before handoff, verify:

- Every spec requirement maps to a task or is explicitly out of scope.
- Failure paths cover backend deps and gate inheritance.
- Acceptance criteria are observable from the entry point.
- Trust-layer work has reachability, runtime probe, registry consult, and destructor/review gates.
- No placeholders or solo architectural pivots.

If review fails, fix the plan before invoking `ultraflow:subagent-driven-development` or `ultraflow:test-driven-development`.
