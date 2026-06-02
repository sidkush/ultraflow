---
name: audit
description: Use every 4-5 commits or before risky continuation when read-only cross-area review is needed.
---

# Audit

## Core Principle

Run a read-only 4+1 audit: four area reviewers write files, then one integrator writes a short summary. Findings require reachability evidence.

## When To Use

- Every 4-5 commits.
- Before continuing after a high-risk trust-layer fold.
- When cumulative drift is suspected.
- When tests pass but operator trust is uncertain.

## Output Locations

Area reports:

`docs/ultraflow/audits/<phase>-<date>-<area>.md`

Integrator:

`docs/ultraflow/audits/<phase>-<date>-SUMMARY.md`

Chat response is only path plus no more than five lines: P0 count, P1 count, top risk, and next action.

## Four Read-Only Areas

1. **trust-layer-wiring:** tenant/auth/audit/scope/path/secrets and reachability.
2. **engine-subsystems:** query/data/agent/runtime subsystems, fallbacks, and recovery.
3. **tests-coverage:** RED/GREEN proof, invocation surface, no live LLM/API tests, baseline drift.
4. **docs-specs:** UFSD, plans, deferred-bugs, rules, operator-comprehension, and rollback docs.

Each area prompt must include:

```markdown
REASONING EFFORT: high - read-only audit across one area
Scope:
Out of scope:
Required: cite file:line or command output for every finding.
Required: Gate A reachability before P0/P1.
Output path:
```

## Integrator

Integrator reads the four reports and writes:

```markdown
# Audit Summary
P0:
P1:
Cross-cutting gaps:
Top risk:
Recommended next action:
Reviewer paths:
```

The integrator may invoke `ultraflow:requesting-code-review` or `ultraflow:adversarial-testing` if a report shows a reachable risk needing deeper review.

## Triage

- P0: stop forward progress.
- P1: fix before next feature/fold.
- P2: fix if scoped or defer with trigger.
- P3/P4: defer or document.

## Anti-Patterns

- Editing code during audit.
- Inlining reports over 30 lines in chat.
- P0/P1 without reachability.
- Creating a separate deferred-bugs sink instead of linking `docs/superpowers/triage/deferred-bugs.md`.
