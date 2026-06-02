---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session.
---

# Subagent-Driven Development

## Core Principle

Fresh implementer per task, then two-stage review: spec compliance first, code quality second. Do not collapse the reviews.

## Entry Gate

Read the plan and UFSD once, then create task tracking. Before dispatch:

- Preflight GO/NO-GO: branch, status, log, test baseline.
- Confirm ticket/Jira binding.
- Confirm branch already exists; do not create worktrees or branches.
- Mark reasoning effort for the run.
- Identify focused tests for each fold. Full floor runs once at the end.

## Per-Task Loop

1. Dispatch implementer with full task text, relevant context, failure modes, and exact working directory. Do not tell subagents to read the plan file.
2. If implementer returns `NEEDS_CONTEXT`, provide missing context once. If still unclear, split task or escalate.
3. If implementer returns `BLOCKED`, change something: context, model tier, decomposition, or plan. Do not retry identically.
4. Run the focused verification yourself.
5. Dispatch spec-compliance reviewer using `spec-reviewer.md`.
6. Fix and re-review until spec compliant.
7. Dispatch code-quality reviewer using `code-quality-reviewer.md`.
8. Fix Critical and Important findings before moving on.
9. Commit the task.

## Long-Dispatch Hygiene

- Focused tests only inside folds.
- Never start the full floor while a reviewer, destructor, or implementer is outstanding.
- Cancel stale long runs when a mid-run fix changes the tested code.
- Non-halting summaries after dispatch and each rebreak round: completed, outstanding, blockers.

## Trust-Layer Branch

For tenant scope, auth gates, audit ledger, scope validator, grounding, path containment, or secrets:

- Runtime probe and HEAD citations before design lock.
- Generic destructor dispatch, not architect-named attackers.
- Reachability check before P0.
- Falsified-findings registry consult.
- Explicit review per commit.

## Review Order Is Mandatory

Spec compliance asks "did we build the requested thing, nothing more and nothing less?" Code quality asks "is it well built?" A clean quality review cannot compensate for missing requirements.

## Completion

After all tasks:

1. Run the full documented floor once.
2. Invoke `ultraflow:requesting-code-review` for the full diff if the change is non-trivial.
3. Invoke `ultraflow:verification-before-completion`.
4. Then use `ultraflow:finishing-a-development-branch` if integration/push is requested.

## Red Flags

- Parallel implementation agents editing shared files.
- Moving to code-quality review before spec compliance passes.
- Trusting implementer test claims without your own command output.
- Starting full suite while work is still changing.
- Proceeding with unaddressed Important review issues.
