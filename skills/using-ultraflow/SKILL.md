---
name: using-ultraflow
description: Use at conversation start. Establishes AskDB UltraFlow skill routing, model routing, preflight discipline, and anti-drift rules.
---

# Using UltraFlow

## Core Stance

UltraFlow is the active methodology namespace. Use `ultraflow:*` skills directly. Superpowers remains an upstream rigor source only; do not instruct agents to invoke `superpowers:*` after this fork.

**Skills are on in both layers:** Opus orchestrates probe, architecture, council, and design lock; Codex executes with TDD, subagents, debugging, review, verification, and git discipline.

## Intent Echo

Before substantial work, state the intended outcome in one sentence. If the user gave a complete dispatch, continue after the echo unless the dispatch contains an unresolved design fork.

## Model Routing

- **Opus:** thinking layer for probes, root cause, architecture, design lock, council, destructor triage, and "which path is live" ambiguity.
- **Codex:** execution layer for TDD, folds, tests, skill authoring, docs, and git. Codex is also the honest gatekeeper: it reports blocked or unverified states plainly.
- **Trust-layer architecture bugs:** route to Opus even if the next action is implementation.
- **Long executions on Opus:** forbidden; they fabricate verification.

Every executor dispatch begins:

`REASONING EFFORT: <none|low|medium|high|xhigh> - <one clause why>`

Default is `medium`. Use `low` for mechanical one-file edits, `high` for multi-file synthesis not probe-mapped, and `xhigh` for trust-layer architecture, deep root cause, or high-ambiguity specs. Raise effort only after improving decomposition.

## Value Check

Before any plan, answer:

1. Does this improve a real operator or user flow?
2. Would we build it if paying an engineer hourly cash?
3. Does it reduce an active bottleneck, trust gap, support burden, retention risk, or differentiation gap?

If no, defer to `docs/superpowers/triage/deferred-bugs.md`. Do not plan.

## Two Orthogonal Axes

**Commit tier = scope and prompt shape.**

| Tier | Shape | Required flow |
|---|---|---|
| T1 | <=5 lines, 1 file, no contradictory evidence | Single autonomous TDD with halt-on-drift |
| T2 | <=30 lines, 1-2 files, edge cases enumerable | Split probe then TDD |
| T3 | Architecture, concurrency, multiple sources of truth, or contradictory evidence | Probe -> destructor -> architect design -> step TDD |

**Finding priority = severity x blast radius.** P0-P4 belongs to adversarial/debug triage. Never merge commit tier with finding priority.

## AskDB Rules To Carry Everywhere

- **Preflight GO/NO-GO:** every executor run checks git status, branch, recent log, and current test baseline. Dirty tree or branch mismatch halts unless the operator explicitly scoped the dirty files.
- **Rule 13 reachability:** before flagging a destructor finding, grep callers and trace input shape from the live entry path. Zero callers or an upstream gate that blocks the bad shape means `SUSPECT-REACHABILITY`.
- **Rule 14 runtime probe:** before design lock, verify call frequency, schema availability, library/runtime semantics, and deployment topology empirically.
- **Rule 21 invocation surface:** probe callers with `<name>(`, subprocess/PATH availability, traversal scope, symlinks, version behavior, and thread/process assumptions. Do not stop at `def <name>`.
- **Falsified registry first:** consult `docs/superpowers/triage/falsified-findings-registry.md` before re-triage.
- **TDD bug fix canonical:** failing test -> RED -> minimal fix -> GREEN + suite -> diff + commit. GREEN-on-arrival is information, not permission to skip.
- **Operator comprehension is correctness:** UUID dropdowns, `[object Object]` banners, hidden controls, and unreadable states are broken even if tests pass.
- **Ship gate:** smoke from the real app entry, real login, happy path, at least one failure mode, clean console, and human-readable labels/errors. Agents mark UI smoke `NOT EXECUTED - deferred for manual verification` unless the operator explicitly asks the execution layer to run it.
- **No live LLM/API in tests:** mock at the network boundary.
- **Jira discipline:** executor runs reference a ticket, comment progress/blockers/SHA/next step, and move status only with evidence.
- **Git discipline:** never auto-create worktrees or branches. Operator manages branches. Skills verify the branch in place. Dirty tree -> halt and ask. On Windows, avoid `&&`, `sed -i`, path-separator traps, and heredoc commit messages; use explicit commands and UTF-8-no-BOM temp files for multiline commits.
- **No calendar estimates:** scope in commits, component count, and decision dependencies.
- **Deferred bugs:** use `docs/superpowers/triage/deferred-bugs.md` as the canonical sink; do not fork it.

## Skill Routing

| Need | Use |
|---|---|
| Creative/design discovery | `ultraflow:brainstorming` |
| Decision between approaches | `ultraflow:council` |
| Multi-step implementation plan | `ultraflow:writing-plans` |
| Implement feature or bugfix | `ultraflow:test-driven-development` |
| Execute independent plan tasks | `ultraflow:subagent-driven-development` |
| Request reviewer pass | `ultraflow:requesting-code-review` |
| Unexpected failure or bug | `ultraflow:systematic-debugging` |
| Independent investigations | `ultraflow:dispatching-parallel-agents` |
| Security/regression pressure | `ultraflow:adversarial-testing` |
| Completion claim or commit gate | `ultraflow:verification-before-completion` |
| Finish branch/PR/merge choice | `ultraflow:finishing-a-development-branch` |
| Code review feedback received | `ultraflow:receiving-review` |
| Periodic 4+1 read-only audit | `ultraflow:audit` |

UI/UX work routes first through taste/impeccable/ui-ux-pro-max/frontend-design, then back to UltraFlow for plan/execution gates.

## Workflow

Opus: probe and gather evidence -> brainstorming -> council -> write spec/plan -> adversarial-testing in SPEC mode -> revise -> hand to Codex.

Codex: preflight -> TDD/subagent execution -> council/adversarial rounds where warranted -> systematic-debugging on RED/GREEN failures -> verification-before-completion -> finishing-a-development-branch.

## UFSD

The canonical UltraFlow State Document location is `docs/ultraflow/specs/`. All skills read and write UFSD entries there. Do not reference legacy non-specs UFSD locations or filename conventions.

## Anti-Drift

Re-read this spine after context compaction, after five tasks, or whenever you notice yourself skipping preflight, TDD, review, or verification. Never trust a subagent's completion claim without your own diff and verification.
