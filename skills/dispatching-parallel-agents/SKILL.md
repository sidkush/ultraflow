---
name: dispatching-parallel-agents
description: Use when facing two or more independent tasks, investigations, or audit areas without shared state or sequential dependency.
---

# Dispatching Parallel Agents

## Core Principle

One focused agent per independent domain. Context is curated; agents do not inherit session history.

## Independence Test

Tasks are independent when they touch different files or read-only areas, do not depend on each other's output, and can be reviewed separately.

Do not parallelize when tasks edit the same files, share mutable state, or require a single root-cause narrative.

## Prompt Contract

Each prompt includes:

- `REASONING EFFORT: <level> - <why>`
- Scope and out-of-scope.
- Required files or pasted context.
- Expected output.
- Confidence score 1-5.
- Reachability requirement for findings.

Score 1-2 -> re-dispatch once with more context. Still 1-2 -> escalate.

## 4+1 Audit Route

For periodic audits every 4-5 commits, use `ultraflow:audit`. It dispatches four read-only area agents:

- trust-layer wiring
- engine subsystems
- tests/coverage
- docs/specs

Then one integrator writes `SUMMARY.md`.

## Long-Dispatch Hygiene

- Dispatch all independent agents in one parallel call.
- Do not start full test floor while agents are outstanding.
- Cancel stale runs when fixes land mid-run.
- After each rebreak round, report completed/outstanding/blockers.

## Integration

Review every result. Resolve conflicts explicitly:

`Conflict: <what differs> | Owner: <why> | Resolution: <final state>`

Then run focused verification and, if appropriate, the final floor.

## Anti-Patterns

- "Fix all tests" as one broad prompt.
- Agents reading broad plan files instead of receiving scoped context.
- Counting unreachable findings as confirmed.
- More than five write agents in parallel.
