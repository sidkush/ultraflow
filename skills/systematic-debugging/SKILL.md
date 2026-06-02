---
name: systematic-debugging
description: Use when encountering any bug, test failure, unexpected behavior, cascade error, or suspected trust gap.
---

# Systematic Debugging

## Iron Laws

1. No fixes before root cause.
2. One bug at a time.
3. No "should fix it"; prove it.

## Session Scope

Maximum three bugs per session. Bug N+1 cannot start until Bug N is RESOLVED or ESCALATED. New bugs discovered mid-flow go to the queue or `docs/superpowers/triage/deferred-bugs.md`.

## Entry

Start with:

- `REASONING EFFORT: <none|low|medium|high|xhigh> - <why>`
- Branch/status/log/test baseline.
- Ticket/Jira reference.
- UFSD read from `docs/ultraflow/specs/`.
- Falsified registry consult if the finding smells familiar.

Choose Simple only for obvious one-file typo/import issues. Auto-promote to Complex on multi-file change, recurrence, CI/env mismatch, or unexpected failure.

## Phase 1: Observe

Reproduce the failure with the exact command. Read the full output. Isolate the smallest trigger. Verify assumptions against HEAD.

Required artifact:

```markdown
Command:
Output:
Expected vs actual:
Assumptions verified:
UFSD sourced:
Recent log:
```

## Phase 2: Reachability And Runtime Probe

Before calling anything P0/P1:

- Grep callers of the suspect code path.
- Trace input shape from real entry point to flagged line.
- If zero callers or an upstream gate blocks the bad shape, mark `SUSPECT-REACHABILITY`.
- Probe runtime assumptions: call frequency, schema availability, library/runtime semantics, deployment topology.
- Probe invocation surface: `<name>(` callers, subprocess PATH, traversal scope, symlinks, version behavior, thread/process model.

## Phase 3: Hypothesize

Create exactly two mechanism-level hypotheses with quoted evidence. Symptom-level hypotheses are rejected.

For each:

```markdown
H1: <component> fails because <precondition> is violated by <caller/condition>. Evidence: <quote>
Counterfactual:
Accepted because:
Prediction:
```

Run prediction-first tests. Both denied -> return to Observe with new evidence.

## Phase 4: Pattern Exhaustion

Extract the broken mechanism and scan for every location sharing it. The fix must cover all reachable locations or explicitly defer them.

## Phase 5: TDD Bug Fix

Follow canonical RED -> GREEN:

1. Write failing regression test for the exact bug.
2. Confirm RED.
3. Minimal fix.
4. Confirm GREEN.
5. Focused regression.
6. Full suite/floor at the end of the fold.

No live LLM/API calls in tests.

## Phase 6: Proof And Skeptic

Proof artifact:

```markdown
Pre-fix behavior:
Post-fix behavior:
Test proof:
Suite/floor proof:
Counterexample:
```

For complex or trust-layer bugs, dispatch a skeptic/root-cause council subset. If two consecutive skeptic runs return not determinable, escalate to the user.

## Journal

Write a debug journal to `docs/ultraflow/debug-journals/YYYY-MM-DD-<slug>.md` for RESOLVED or ESCALATED bugs. Link deferred follow-ups to `docs/superpowers/triage/deferred-bugs.md`; do not create a separate deferred journal.

## Loop Detector

Fire council escalation when the same hypothesis or semantic diff repeats in a second session, or after three sessions without sign-off. Do not recursively council inside a council-spawned debug session.

## Red Flags

- Fix before root cause.
- CONFIRMED without prior prediction.
- P0 without reachability.
- Design lock without runtime probe.
- Probing `def <name>` but not callers/invocation surface.
- Hiding errors with catch-all handling.
- `[object Object]` or incomprehensible operator state treated as UI polish rather than correctness.
