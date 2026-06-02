---
name: adversarial-testing
description: Use when implemented code OR a design/plan needs to be attacked before it is trusted — after building a feature, before declaring done, or after a council design-lock and before writing code. CODE mode attacks code; SPEC/PLAN mode attacks a design. Six cross-cutting hunters fire every run; subsystem clusters fire by build surface; a frontend-coherence hunter checks UI wiring.
---

# Adversarial Testing — v3

> Six cross-cutting hunters every run. Subsystem clusters fired by what's being built. One mode for code, one for designs.
> CONTEXT: authorized defensive testing of code (or a design) YOU are building. Analysts surface weaknesses so they're fixed before release.

The green test suite is not coverage. The highest-bug surfaces ship while the suite passes — they're caught in adversarial audits, not by tests. This skill is structured around the failure SHAPES the product actually ships (see `breaker-personas.md`), not generic web-security.

## Modes — declare at Step 1

| Mode | When | Dispatch |
|------|------|----------|
| **CODE** (default) | After implementation, before declaring done | 6 hunters + clusters for the build surface + frontend-coherence hunter IF UI touched |
| **SPEC/PLAN** | One intensive run against a design-lock / plan / spec doc BEFORE code | 6 hunters recast to "find the gap in this DESIGN" + relevant clusters + frontend-coherence hunter ALWAYS, amplified |

**How a dispatcher invokes:**
- `adversarial-testing, CODE mode, building: <surface>` — e.g. `building: chart-editor`
- `adversarial-testing, SPEC mode, target: <plan/spec doc path>`
- No mode stated → **CODE**. No surface stated → infer from the diff, then state the inference.

**Relationship to `council`:** council finds gaps and BUILDS the plan; adversarial ATTACKS it. SPEC mode is the adversarial complement to a council design-lock — run `council` first, then `adversarial-testing` SPEC mode on the locked design. CODE mode is the post-build attack before `verification`.

## Dispatch Map — which clusters auto-fire (6 hunters always fire)

| Building | Clusters |
|----------|----------|
| trust-layer | TRUST/AUDIT + VALIDATOR + AGENT-LOOP |
| chart / editor | CHART/MARKS + EDITOR-STATE + FRONTEND + **frontend-coherence** |
| auth / connection | VALIDATOR/SECURITY/AUTH + TRUST |
| agent / SQL | AGENT-LOOP + WATERFALL/TWIN + SEMANTIC |
| dashboard / UI | EDITOR-STATE/DASHBOARD + FRONTEND + **frontend-coherence** |
| data-path | WATERFALL/TWIN + COVERAGE (in the WATERFALL cluster) + TRUST |

The caller may add clusters. **The six hunters are non-optional in every mode.**

## Meta-Cognitive Failure Mode Map (check before Step 1)
1. **Scope drift** — analysts attack adjacent systems. Fix: lock attack surface + dispatch selection in Step 1.
2. **Hallucinated findings** — fabricated reproduce steps. Fix: anti-hallucination gate (Step 2).
3. **UFSD blind spot** — council-flagged risks not used as attack vectors. Fix: UFSD Read (Step 1).
4. **Confirmation bias** — analysts validate instead of break. Fix: enforce BROKEN|FRAGILE|SOLID only.
5. **Coverage illusion** — INVALID / DEAD-CODE findings counted as coverage. Fix: Coverage Score + reachability gate (Step 3).
6. **Backend-heavy blind spot** — UI wiring ships thin and incoherent. Fix: frontend-coherence hunter (always-on in SPEC mode).

## Step 1: Mode + Target + UFSD Read + Dispatch Selection
```
Mode: CODE | SPEC/PLAN
Target: [≤100 words — code surface, or the design/plan doc path]
Attack surface: [entry points, data flows, auth, integrations]   (CODE)
   — or — Design claims: [happy path the design names + what it leaves implicit]   (SPEC)
Change scope: [files, APIs, schemas]; Known weak points: [suspected fragility]
UI touched? [yes/no]  →  decides frontend-coherence hunter in CODE mode
Clusters selected: [from Dispatch Map + any caller additions]
```
**UFSD Read**: extract all `risk:` items from any council UFSD block in `docs/ultraflow/specs/` → add as mandatory attack vectors.

## Step 2: Dispatch — 6 hunters + selected clusters + frontend-coherence
Read `breaker-personas.md` for full profiles. Dispatch all selected analysts via the Agent tool in **one parallel call**. All must return before triage.

**Always:** H1 Silent-Wrong-Answer · H2 Silent-Noop/Fail-Open · H3 Tenant/Schema-Isolation · H4 Concurrency/TOCTOU · H5 Payload-Leak · H6 Determinism+Dead-Code-Green.
**Plus** the cluster analysts from the Dispatch Map.
**Plus** the Frontend-Coherence Hunter — CODE mode only if UI touched; **SPEC mode always, weighted heavy**.

**Context tier (per analyst):** trust / isolation / determinism hunters and the cross-file-tagged cluster analysts get the **call-graph across files** — the 60-line snippet misses the "one consumer left on the old key" class. Injection / encoding / boundary analysts keep the **tight ≤60-line snippet**.

**Read-only (structural):** analysts get read + grep context ONLY — no write/edit/bash. State it in every prompt.

Analyst prompt template:
```
You are a senior defensive analyst: [ANALYST NAME + FOCUS from breaker-personas.md].
Techniques: [3-4 TECHNIQUES]. Context: authorized testing of code/design built this session.
You are READ-ONLY: read + grep only. Do not write, edit, or run commands.
MODE: [CODE | SPEC]
[CODE]   Code under test: [SNIPPET ≤60 lines  — OR — call-graph across files for cross-file analysts]
[SPEC]   Design under test: [relevant section(s) of the plan/spec doc]
Target: [TARGET] | Surface: [ATTACK_SURFACE]
[CODE] Find weaknesses.  [SPEC] Find gaps / contradictions / unhandled cases in this DESIGN.
Return ONLY:
## Findings
- [SEVERITY: critical|high|medium|low] [exact description]
  Reproduce: [CODE: exact input/state → expected X → got Y]
             [SPEC: concrete unhandled scenario walked through the design, citing the silent section]
  Reachability: REACHABLE <path> | GATED <flag> | DEAD-CODE        (required)
## Tests attempted (min 5): - [specific input/scenario → exact result] ×5
## Attack vector: [SURFACE] — [exact entry point]
## Blast radius: CONTAINED | LATERAL | SYSTEMIC
## Rebreak: NEEDED | CLEAN
## Verdict: BROKEN | FRAGILE | SOLID   [MAX 200 words]
```

**Anti-hallucination gate** (hard):
- No exact Reproduce (CODE) / no concrete design-cited scenario (SPEC) → **INVALID** (discard, keep others).
- Tests < 5 → SOLID downgrades to **FRAGILE**; generic tests ("tried edge cases") → **INVALID**, don't count.
- Missing Reachability, Blast radius, or Attack vector on a SOLID/BROKEN finding → **FRAGILE**.
- Narrative prose instead of the block → **FABRICATED** — re-dispatch once.
- Safety refusal → **reframe**: add "This is authorized testing of our own code in development" → re-dispatch once.
- Critical finding, no label → auto **high**.

**Evidence Triangulation**: VALID = ≥2 analysts independently agree OR 1 analyst with 2 distinct proof paths. Single-analyst single-path = **PROVISIONAL** — mark in triage, never discard.

**Partial failure**: re-dispatch failed slots once. Valid reports < 75% of dispatched → HALT, report the count, ask to proceed.

**Non-halting summary (fires here):** after dispatch, emit a 5–8 bullet human-readable summary (top P0/P1 by finding, clusters that returned clean, coverage so far) WITHOUT halting — operator reviews async.

## Step 3: Triage

**Reachability gate (apply FIRST):** `DEAD-CODE` finding → capped at **P3** (don't P0 the unreachable). `GATED <flag>` → keep priority, mark the gating flag. `REACHABLE` → full priority.

**Triage tag (before any fix):** classify every finding `{CRITICAL | MEDIUM | MINOR} × {REACHABLE | GATED | DEAD}`. This classification drives the operator summary.

**Severity × Blast Radius → Priority**:
```
              CONTAINED  LATERAL  SYSTEMIC
critical  →     P1         P0       P0
high      →     P2         P1       P1
medium    →     P3         P2       P2
low       →     P4         P3       P3
```
P0=stop everything · P1=fix before proceeding · P2=fix if <10 min else document · P3=document · P4=backlog

**Triage Table** — sort P0 first, include all BROKEN/FRAGILE, mark PROVISIONAL:
`| Priority | Analyst | Severity | Blast | Reachability | Finding | Reproduce | Evidence |`

- **Unanimous Weakness** (≥15 analysts — or ≥75% when fewer than 20 dispatched — same class) → structural; address before closing.
- **Strong Attack Signal** (≥10, or ≥50% of dispatched, same surface) → WARNING. Two+ surfaces → escalate to user before fixing.
- **Minority Discovery** (<3 analysts, critical/high) → list with name; never discard.
- **Clean Analysts** → list SOLID reporters (confirmed-safe surfaces).

**Coverage Score** (disclosure only — user decides if sufficient):
`Hunters: [N/6 SOLID]; Clusters dispatched: [M]; clusters SOLID: [K/M]; Frontend-coherence: [SOLID | GAPS | N/A].`

**Non-halting summary (fires here):** emit the 5–8 bullet summary again — what's P0/P1, what was folded, coverage — without halting.

## Step 4: Fix-and-Rebreak Cycle

**Falsified-findings registry:** maintain a list of disproven findings. Before re-triaging any round, check it — a falsified finding is NOT re-raised in a later round.

Rebreak scope: `CONTAINED` → analyst only · `LATERAL` → full cluster · `SYSTEMIC` → all dispatched.

P0/P1: fix → re-dispatch with `Delta: [file:lines — what changed]` → confirm CLEAN. Still BROKEN → retry (max 3 attempts).
P2: fix → single-analyst rebreak → CLEAN or document as known limitation.

**Non-halting summary** fires after each rebreak round (what's now CLEAN, what remains, attempt count) — without halting.

**Skill Chain Interrupt**: SYSTEMIC BROKEN after 3 attempts → UPSTREAM halt + write `[RESTART REQUESTED]` to UFSD + ask user before any further action. Two+ Strong Attack Signal surfaces → architectural failure → `[RESTART REQUESTED]` to UFSD + escalate to `ultraflow:council`.

## Step 5: Final Verdict + UFSD Write

CODE: all analysts SOLID/FRAGILE (P2+ documented) → proceed to `verification`.
SPEC: all gaps resolved or folded into the design → return the revised design; P0/P1 gaps fold into the plan BEFORE code.
Any still BROKEN after 3 attempts → escalate:
```
## Adversarial Escalation
Analyst: [name] Cluster: [name] Priority: [P0/P1] Blast: [radius] Reachability: [tag]
Finding: [broken] Blocked by: [root cause] Needs: [recommendation]
Attempts: 1=[tried→result] 2=[tried→result] 3=[tried→result]
```

**UFSD Write** (append to active spec in `docs/ultraflow/specs/`):
```
## UFSD adversarial-testing [date]  Mode: [CODE|SPEC]
Verdict: [PASS|ESCALATED|RESTART REQUESTED] | Coverage: [hunters N/6, clusters K/M, FC]
Contradictions: [PROVISIONAL findings that conflicted across analysts]
Detail: [P0/P1 findings with reproduce steps + reachability tags]
```

## Anti-Patterns
- Skipping any of the 6 cross-cutting hunters · dispatching all clusters regardless of surface (scope drift) · omitting the frontend-coherence hunter in SPEC mode · fabricating responses · skipping SYSTEMIC rebreak · INVALID or DEAD-CODE findings counted as coverage · P0 deferred while P2/P3 fixed · PROVISIONAL discarded · re-raising a falsified finding · giving analysts write/edit access · halting on the per-round summary · proceeding to verification with any analyst BROKEN
