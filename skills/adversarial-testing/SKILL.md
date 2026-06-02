---
name: adversarial-testing
description: Use when implemented code or a design/plan needs authorized defensive pressure before it is trusted.
---

# Adversarial Testing v3

Six cross-cutting hunters fire every run. Subsystem clusters fire by surface. SPEC/PLAN mode attacks a design before code; CODE mode attacks the diff; TRUST mode performs generic destructor dispatch for trust-layer paths.

The green test suite is not coverage. The highest-risk surfaces ship while tests pass; adversarial testing targets those failure shapes.

## Shared Council Gates

Use this exact wording, identical to `ultraflow:council`:

**GATE A - Reachability/Proof-of-Life:** a persona names the live entry path by grep/trace to the targeted code; dead code, non-existent APIs, or unreachable inputs cannot be CONFIRMED.

**GATE B - Frontend-Wiring/Coherence:** a persona names the frontend seam that consumes the backend change, confirms payload-shape match, and accounts for loading, error, empty, stale, and agent-editability states.

Any finding without Gate A is `SUSPECT-REACHABILITY`, not P0/P1. Any backend/UI contract finding without Gate B is PROVISIONAL until the seam is traced.

## Modes

| Mode | When | Dispatch |
|---|---|---|
| CODE | After implementation, before completion | 6 hunters + clusters for touched surface + frontend-coherence if UI touched |
| SPEC/PLAN | After council/design lock, before code | 6 hunters recast to attack the design + relevant clusters + frontend-coherence always |
| TRUST | Trust-layer path or security-sensitive finding | Generic destructor dispatch; architect gives context but never names attack angles |

Invocation examples:

- `adversarial-testing, CODE mode, surface: chart-editor`
- `adversarial-testing, SPEC mode, target: docs/ultraflow/specs/...`
- `adversarial-testing, TRUST mode, surface: auth/audit/scope`

Council chooses or validates a direction. Adversarial attacks it. Run `ultraflow:council` before SPEC mode when the design is not locked.

## Dispatch Map

| Surface | Clusters |
|---|---|
| trust-layer | TRUST/AUDIT + VALIDATOR + AGENT-LOOP |
| chart/editor | CHART/MARKS + EDITOR-STATE + FRONTEND + frontend-coherence |
| auth/connection | VALIDATOR/SECURITY/AUTH + TRUST |
| agent/SQL | AGENT-LOOP + WATERFALL/TWIN + SEMANTIC |
| dashboard/UI | EDITOR-STATE/DASHBOARD + FRONTEND + frontend-coherence |
| data-path | WATERFALL/TWIN + COVERAGE + TRUST |

The caller may add clusters. The six cross-cutting hunters are non-optional in every mode.

## Failure Mode Map

1. Scope drift: lock attack surface and selected clusters.
2. Hallucinated findings: require exact reproduce or design-cited scenario.
3. UFSD blind spot: read risks from `docs/ultraflow/specs/`.
4. Confirmation bias: force BROKEN/FRAGILE/SOLID verdicts.
5. Coverage illusion: INVALID or dead-code findings do not count.
6. Backend-heavy blind spot: frontend-coherence is always-on in SPEC mode.

## Step 1: Mode, Target, UFSD, Dispatch Selection

```markdown
Mode: CODE | SPEC/PLAN | TRUST
Target:
Attack surface:
Design claims:
Change scope:
UI touched:
Clusters selected:
Gate A required: yes|no
Gate B required: yes|no
```

Read the active UFSD from `docs/ultraflow/specs/` and add council `risk:` items as mandatory attack vectors.

## Step 2: Dispatch

Read `breaker-personas.md`. Dispatch:

- Always: H1 Silent-Wrong-Answer, H2 Silent-Noop/Fail-Open, H3 Tenant/Schema-Isolation, H4 Concurrency/TOCTOU, H5 Payload-Leak, H6 Determinism+Dead-Code-Green.
- Plus clusters from the dispatch map.
- Plus Frontend-Coherence in CODE when UI is touched, and always in SPEC/PLAN.

Context tier:

- Cross-file call graph for isolation, correctness, determinism, and trust analysts.
- Tight <=60-line snippets for injection, encoding, and boundary analysts.

Analysts are read-only: read + grep only. No write/edit/bash.

Analyst prompt:

```markdown
You are a senior defensive analyst: <name and focus from breaker-personas.md>.
Techniques: <3-4 techniques>.
Context: authorized testing of code/design built this session.
You are READ-ONLY: read + grep only. Do not write, edit, or run commands.

MODE: CODE | SPEC | TRUST
Target:
Surface:
Code/design context:

Return ONLY:
## Findings
- Severity: critical|high|medium|low
  Description:
  Reproduce:
  Gate A - Reachability/Proof-of-Life: PASS|FAIL|N/A - <evidence>
  Gate B - Frontend-Wiring/Coherence: PASS|FAIL|N/A - <evidence>
  Reachability: REACHABLE <path> | GATED <flag> | DEAD-CODE
  Tests attempted: <min 5 exact inputs/scenarios>
  Attack vector:
  Blast radius: CONTAINED | LATERAL | SYSTEMIC
  Recovery path:
  Rebreak: NEEDED | CLEAN
  Verdict: BROKEN | FRAGILE | SOLID
```

Anti-hallucination gate:

- No exact CODE reproduce or SPEC design-cited scenario -> INVALID.
- Fewer than five concrete attempts -> SOLID downgrades to FRAGILE; generic attempts are INVALID.
- Missing reachability, blast radius, or attack vector -> FRAGILE.
- Narrative prose instead of the block -> re-dispatch once.
- Safety refusal -> reframe as authorized testing of our own code and re-dispatch once.
- Critical without label -> auto high.

Evidence triangulation: VALID means at least two analysts independently agree, or one analyst has two distinct proof paths. Single-analyst, single-path findings are PROVISIONAL, never discarded silently.

Emit a non-halting 5-8 bullet summary after dispatch.

## Step 3: Triage

Reachability first:

- DEAD-CODE caps at P3.
- GATED keeps priority but names the gating flag.
- REACHABLE can be P0/P1.

Before P0:

1. Gate A must pass.
2. If frontend/operator surface is involved, Gate B must pass.
3. Check `docs/superpowers/triage/falsified-findings-registry.md`.

Priority matrix:

```text
              CONTAINED  LATERAL  SYSTEMIC
critical         P1        P0        P0
high             P2        P1        P1
medium           P3        P2        P2
low              P4        P3        P3
```

Triage tag every finding as `{CRITICAL|MEDIUM|MINOR} x {REACHABLE|GATED|DEAD}` before fixing.

Coverage score:

`Hunters: N/6 SOLID; Clusters dispatched: M; clusters SOLID: K/M; Frontend-coherence: SOLID|GAPS|N/A`

## Step 4: Fix And Rebreak

Maintain a falsified-findings list. Re-check it before re-triage.

Rebreak scope:

- CONTAINED: original analyst.
- LATERAL: full cluster.
- SYSTEMIC: all dispatched.

P0/P1: fix before proceeding, rebreak, retry up to three attempts. Still BROKEN after three attempts -> write `[RESTART REQUESTED]` to UFSD and escalate to `ultraflow:council`.

P2: fix if cheap and scoped; otherwise document and defer.

P3/P4: document or defer to `docs/superpowers/triage/deferred-bugs.md`.

Emit non-halting summaries after every rebreak round.

## Step 5: Final Verdict And UFSD Write

CODE: all analysts SOLID/FRAGILE with P2+ documented -> proceed to `ultraflow:verification-before-completion`.

SPEC/PLAN: all gaps resolved or folded into the design before code.

Append to active UFSD:

```markdown
## adversarial-testing - <date>
Mode:
Verdict:
Gate A summary:
Gate B summary:
Coverage:
P0/P1 findings:
Deferred:
Recovery paths:
```

## Anti-Patterns

- Skipping any cross-cutting hunter.
- Dispatching all clusters regardless of surface.
- Omitting frontend-coherence in SPEC mode.
- Architect-named attackers in TRUST mode.
- P0 without reachability.
- Counting INVALID or DEAD-CODE findings as coverage.
- Re-raising a falsified finding.
- Giving analysts write/edit access.
- Proceeding to verification with BROKEN P0/P1 findings.
