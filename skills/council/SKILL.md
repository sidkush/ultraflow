---
name: council
description: Use when a decision needs multi-perspective review, architecture selection, trust-layer design, or root-cause escalation.
---

# Council v2

## Core Principle

Council confirms decision themes only when evidence proves the code path is live and the frontend/operator surface is coherent. Vote count is secondary to reachability and wiring.

## When To Use

- Architecture or implementation approach choice.
- Trust-layer design lock.
- Stuck systematic-debugging loop.
- Plan adversarial SPEC mode.
- Any decision with conflicting evidence or meaningful rollback cost.

## Preflight

Read active UFSD from `docs/ultraflow/specs/`. Carry `[LOCKED]` decisions forward. Start with:

`REASONING EFFORT: high - council decision requires evidence synthesis`

If the decision is trust-layer or high ambiguity, use `xhigh`.

## Mandatory Gates

These exact gate labels are shared with `ultraflow:adversarial-testing`:

**GATE A - Reachability/Proof-of-Life:** a persona names the live entry path by grep/trace to the targeted code; dead code, non-existent APIs, or unreachable inputs cannot be CONFIRMED.

**GATE B - Frontend-Wiring/Coherence:** a persona names the frontend seam that consumes the backend change, confirms payload-shape match, and accounts for loading, error, empty, stale, and agent-editability states.

A theme cannot move from PROVISIONAL to CONFIRMED until both gates pass on the record. If a gate is irrelevant, record why as `N/A` with evidence.

## Always-On Lenses

Dispatch these lenses every decision run:

1. Reachability/Proof-of-Life gate.
2. Frontend-Wiring/Coherence gate.
3. Determinism.
4. Trust correctness.
5. Trust contention.
6. Trust durability.
7. Contrarian/minority dissent.
8. Distributed systems/event ordering.
9. Paint/DOM frontend.

## Decision-Type Clusters

Add only the relevant cluster lenses:

| Decision type | Extra lenses |
|---|---|
| trust/isolation | tenant scope, auth, audit, secrets, file boundary |
| agent-loop/SQL-gen | prompt/tool loop, SQL validator, scope validator, synthesis trust |
| chart/worksheet-spec | visual spec, field pills, chart IR, operator editability |
| data-path/waterfall | schema, memory, turbo twin, live tier, provenance |
| frontend-UX | state inheritance, labels, failure UX, accessibility |
| semantic-layer | metrics, dimensions, lineage, aggregation trust |

## Persona Dispatch

Use `personas.md`. Dispatch relevant personas by lens, not decorative roles. The two gate personas are mandatory. Include Contrarian even for apparent consensus.

Each persona returns:

```markdown
## Persona
Name:
Lens:
Evidence:
Gate A: PASS|FAIL|N/A - <file:line or command>
Gate B: PASS|FAIL|N/A - <file:line or command>
Recommendation:
Recovery path:
Assumptions:
Confidence: 1-5
```

Evidence must cite live files, commands, or probe output. Generic "best practice" evidence is invalid.

## Synthesis

Cluster by mechanism, not wording. For each theme record:

```markdown
| Theme | Mechanism | Personas | Gate A | Gate B | Status | Lead Risk | Recovery | Dissent |
```

Status rules:

- **CONFIRMED:** at least two independent personas support the mechanism, Gate A passes, Gate B passes or is evidence-backed N/A, and recovery is named.
- **PROVISIONAL:** evidence is promising but a gate, recovery path, or independent confirmation is missing.
- **REJECTED:** unreachable, incoherent frontend seam, contradicted by evidence, or unbounded recovery.

Do not ship PROVISIONAL as CONFIRMED.

## Required Synthesis Sections

- Theme table.
- Unanimous concerns.
- Minority report.
- Contradiction map.
- Assumption registry.
- Counterfactual gate.
- Recovery path for fail-closed decisions.
- Sequencing by decision leverage: cheap gates before expensive builds.
- Coverage score and uncovered lenses.

## UFSD Write

Append to `docs/ultraflow/specs/<active UFSD>`:

```markdown
## council - <date>
Decision:
Theme status:
Gate A:
Gate B:
Recovery path:
Assumptions:
Counterfactual:
```

## User Selection

Ask the user to choose, combine, or revise when multiple viable themes remain. Do not infer preference from earlier conversation if the council surfaced new evidence.

## Relationship To Adversarial

Council chooses or validates a direction. `ultraflow:adversarial-testing` attacks the chosen direction or plan. Both share Gate A and Gate B wording so a plan cannot bypass reachability or frontend coherence.

## Anti-Patterns

- Confirming themes without both gates.
- Treating vote count as truth.
- Letting generic personas replace live evidence.
- Detection without recovery.
- Proceeding to `ultraflow:writing-plans` with unresolved factual contradictions.
