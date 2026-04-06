---
name: planning
description: Use when spec/requirements exist, before touching code. Generates implementation plans with precise task granularity, file paths, test commands, and git steps. Self-reviews via plan-reviewer subagent.
---

# Streamlined Planning

## When to Use
- After discovery produces an approved spec
- After council selects an approach
- Before ANY implementation begins
- Skip only for single-file, single-function changes where the change is obvious

## Pre-Planning Protocol

### 1. UFSD Read
Read the UFSD summary file (`docs/ultraflow/specs/UFSD.md`). Extract:
- **Approach**: chosen solution from council
- **Scope**: exact boundaries of what is in/out
- **Assumptions**: upstream assumptions logged by discovery/council
- **Invariants**: system invariants that must not be broken

Do NOT re-discover these. Carry them forward.

### 2. Meta-Cognitive Failure Mode Map
Before writing the plan, generate 5 failure modes specific to THIS planning session. Format:

```
Failure Mode Map — [Feature Name]
1. [FM-1]: [What could go wrong in the plan itself]
2. [FM-2]: ...
3. [FM-3]: ...
4. [FM-4]: ...
5. [FM-5]: ...
```

Inject this list into the plan-reviewer subagent prompt so it checks for each failure mode explicitly.

### 3. Assumption Registry
Log every new planning assumption to the UFSD detail section. Each entry:
- `ASSUMPTION: [statement]` — validated by [evidence] OR `[UNVALIDATED — risk item]`

Unvalidated assumptions must appear as risk items in the plan.

### 4. Invariant Preservation
List all system invariants extracted from UFSD. Example invariants: data integrity constraints, API backward-compatibility, auth boundaries, idempotency guarantees.

Each invariant is labeled `Invariant-N: [statement]`. Tasks that could violate an invariant must include an explicit invariant-check test step.

## Plan Structure

Write plan to `docs/ultraflow/plans/YYYY-MM-DD-<feature>.md`:

```markdown
# Plan: [Feature Name]
**Spec**: [link to spec file]
**UFSD**: [link to UFSD summary]
**Approach**: [chosen council approach or direct description]
**Branch**: `feature/<name>`

## Assumption Registry
- ASSUMPTION: [statement] — [validated / UNVALIDATED — risk item]

## Invariant List
- Invariant-1: [statement]
- Invariant-2: [statement]

## Failure Mode Map
1. [FM-1]: ...  2. [FM-2]: ...  3. [FM-3]: ...  4. [FM-4]: ...  5. [FM-5]: ...

## Tasks
### Task 1: [Name] (~N minutes)
- **Files**: `path/to/file.ext` (create|modify)
- **Intent**: [What this code does — NOT full code blocks]
- **Invariants**: [Invariant-N it could affect, or "none"]
- **Assumptions**: [logged to UFSD / none]
- **Test**: `[exact command]` → expects [specific output]
- **Invariant-Check**: `[command]` → confirms [invariant] holds  ← only if Invariants field is non-empty
- **Commit**: `[commit message]`

### Task 2: ...

## Scope Validation
Tasks in scope: [list]
Tasks flagged: [SCOPE DEVIATION — requires approval] or "none"

## Counterfactual Gate
Strongest argument AGAINST this plan: [X — cite evidence or reasoning].
We accept this plan because: [Y — cite evidence or reasoning].
> Impact estimates are REASONED, not PROVEN — assumption chain: [list assumptions underpinning any estimate].

## MVP-Proof
Claims about performance or scalability: [claim → evidence citation, or "none"].

## Fingerprint
[1-line summary of what this plan produces when complete]
```

## Rules

1. **2-5 minute tasks**: Each task should take a subagent 2-5 minutes. Split larger tasks.
2. **Code intent for simple sections**: Describe WHAT the code does, not the full implementation. Saves ~50% tokens.
3. **Code blocks for critical sections**: Auth logic, data models, API contracts, security checks, and migration schemas MUST include actual code blocks to prevent implementation divergence across subagents.
4. **Exact file paths**: Every task must name exact files to create or modify.
5. **Test commands with expected output**: Not just "run tests" but the exact command and what success looks like.
6. **Task independence**: Mark dependencies explicitly. Independent tasks can run in parallel.
7. **Commit per task**: Each task gets its own commit message.
8. **Fingerprint**: Every plan ends with a 1-line fingerprint summarizing the expected end state. When referenced later, compare current state against fingerprint to detect staleness.
9. **Scope validation**: Every task is checked against UFSD scope. Out-of-scope tasks are flagged `[SCOPE DEVIATION — requires approval]` and blocked until approved.
10. **Invariant-check test**: Any task touching a listed invariant must include an invariant-check test step in addition to its normal test step.
11. **MVP-Proof**: Performance or scalability claims must cite evidence. Estimates are REASONED, not PROVEN — include the assumption chain.

## Counterfactual Gate

Before finalizing the plan, complete this gate (written into the plan under "Counterfactual Gate"):

- **Counterfactual**: "The strongest argument against this plan is [X]." X must reference a specific failure mode, assumption gap, or architectural risk — not a generic concern.
- **Acceptance**: "We accept because [Y]." Y must cite evidence (benchmark, precedent, spec requirement) or a validated assumption chain.

Do not proceed to self-review until both sides are documented.

## Self-Review

After writing the plan, dispatch the `plan-reviewer` subagent (read `plan-reviewer.md` in this directory for the prompt). Pass the Failure Mode Map to the reviewer. The reviewer checks:

- [ ] Every spec requirement is covered by at least one task
- [ ] No task contains "TBD", "TODO", "etc.", "similar to Task N", or vague language
- [ ] All file paths exist or are marked as "create"
- [ ] Test commands are executable (not pseudocode)
- [ ] Task order respects dependencies
- [ ] Names are consistent (same function/variable name throughout)
- [ ] Each task is ≤10 minutes of estimated work
- [ ] Each task has a commit message
- [ ] Plan ends with a Fingerprint line
- [ ] Each of the 5 failure modes is either mitigated by a task or acknowledged as accepted risk
- [ ] Counterfactual Gate is complete with evidence on both sides
- [ ] All SCOPE DEVIATION flags have approval or are removed
- [ ] Every invariant-touching task has an invariant-check test step
- [ ] All UNVALIDATED assumptions are listed as risk items

If review fails → fix and re-review. Do NOT proceed to building with review failures.

When plan passes review → invoke `building`. For independent tasks → note which for `parallel-dispatch`.

## UFSD Write

After the plan passes review, update the UFSD:
- **Summary**: append one line — `[YYYY-MM-DD] Planning complete. Branch: feature/<name>. Fingerprint: <fingerprint>.`
- **Detail**: append the full Assumption Registry, Invariant List, Failure Mode Map, and Counterfactual Gate from the plan.

This ensures downstream skills (building, verification) inherit the full planning context from UFSD without re-reading the plan file.

## Anti-Patterns
- Writing full code blocks in plans (wastes tokens, gets stale)
- Tasks longer than 10 minutes (too coarse for subagents)
- "Test it works" without specifying HOW
- Skipping self-review ("the plan looks fine")
- Skipping the UFSD Read and re-discovering scope/assumptions from scratch
- Counterfactual Gate with generic concerns instead of specific failure modes
- Logging assumptions without marking unvalidated ones as risk items
