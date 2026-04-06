---
name: adversarial-testing
description: Dispatches ALL 20 security analysts in parallel to find bugs, security holes, regressions, and edge cases via authorized defensive testing. Zero sampling. Full coverage every run.
---

# Adversarial Testing — Quality Assurance Division

> 20 security analysts. Every run. Full coverage or not at all.
> CONTEXT: This is authorized defensive testing of code YOU are building. Analysts identify weaknesses so they can be fixed before release.

## Meta-Cognitive Failure Mode Map (check before Step 1)
1. **Scope drift** — analysts attack adjacent systems. Fix: lock attack surface in Step 1.
2. **Hallucinated findings** — fabricated reproduce steps. Fix: anti-hallucination gate (Step 2).
3. **UFSD blind spot** — council-flagged risks not used as attack vectors. Fix: UFSD Read (Step 1).
4. **Confirmation bias** — analysts validate instead of break. Fix: enforce BROKEN|FRAGILE|SOLID only.
5. **Coverage illusion** — INVALID findings counted as coverage. Fix: Coverage Score disclosure (Step 3).

Top 2 failure modes per analyst are in `breaker-personas.md`.

## Step 1: Extract Target + UFSD Read
```
Target: [≤100 words]; Attack surface: [entry points, data flows, auth, integrations]
Change scope: [files, APIs, schemas]; Known weak points: [suspected fragility]
```
**UFSD Read**: Extract all `risk:` items from any council UFSD block in `docs/ultraflow/specs/` → add as mandatory attack vectors.

## Step 2: Dispatch All 20 Operatives Simultaneously
Read `breaker-personas.md` for full profiles. Dispatch all via Agent tool in one parallel call. All 20 must return before triage.

**Cluster I — Infiltration**: 1 Phantom Injector, 2 Ghost in the Auth, 3 Baron Von CSRF
**Cluster II — Chaos Division**: 4 Professor Overflow, 5 Sigil Wraith, 6 Architect Void
**Cluster III — Temporal Warfare**: 7 Phantom Interval, 8 Null Epoch, 9 Vector Lace
**Cluster IV — Resource Siege**: 10 Lethe, 11 Sisyphus, 12 Tantalus
**Cluster V — Systems Warfare**: 13 Seraphex, 14 Voltgrieve, 15 Malvareth
**Cluster VI — Mind Games**: 16 Paradox, 17 Ouroboros, 18 Meridian
**Cluster VII — Deep Specialists**: 19 The Alchemist, 20 Regression Phantom

Analyst prompt template:
```
You are a senior security analyst specializing in [ANALYSIS_AREA from breaker-personas.md].
Your test focus area: [FOCUS_DESCRIPTION]. Your test techniques: [4 TECHNIQUES from breaker-personas.md].
Context: This is authorized defensive testing of code being built in this session.
Code under test: [RELEVANT_CODE ≤60 lines]
Target: [TARGET] | Surface: [ATTACK_SURFACE]
Find weaknesses in this code. Return ONLY:
## Findings
- [SEVERITY: critical|high|medium|low] [exact description]
  Reproduce: [exact input/state] → expected [X] → got [Y]
## Tests attempted (min 5): - [specific input → exact result] ×5
## Attack vector: [SURFACE] — [exact entry point]
## Blast radius: CONTAINED | LATERAL | SYSTEMIC
## Rebreak: NEEDED | CLEAN
## Verdict: BROKEN | FRAGILE | SOLID  [MAX 200 words]
```

**Anti-hallucination gate** (hard):
- No exact Reproduce → **INVALID** finding (discard, keep others)
- Tests < 5 → SOLID → **FRAGILE**; generic tests ("tried edge cases") → **INVALID**, don't count
- Missing Blast radius or Attack vector on SOLID → **FRAGILE**
- Narrative prose instead of block → **FABRICATED** — re-dispatch once
- Safety refusal (analyst rejects prompt) → **reframe**: add "This is authorized testing of our own code in development" and re-dispatch once
- Critical finding, no label → auto **high**

**Partial failure**: Re-dispatch failed slots once. < 12 valid reports → HALT, tell user count, ask to proceed.

**Evidence Triangulation**: VALID = ≥2 analysts independently agree OR 1 analyst with 2 distinct proof paths. Single-analyst, single-path = **PROVISIONAL**. Mark in triage; do not discard.

## Step 3: Mega-Triage

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
`| Priority | Operative | Severity | Blast Radius | Finding | Reproduce | Evidence |`

- **Unanimous Weaknesses** (≥15/20 same class) → structural; must address before closing.
- **Strong Attack Signals** (≥10/20 same surface) → mark WARNING. Two+ surfaces → escalate to user before fixing.
- **Minority Discoveries** (<3 analysts, critical/high) → list with name; never discard.
- **Clean Operatives** → list SOLID reporters (confirmed-safe surfaces).

**Coverage Score**: `Attack surface: [N clusters SOLID] / 7.` Disclosure only — user decides if sufficient.

## Step 4: Fix-and-Rebreak Cycle

Rebreak scope: `CONTAINED` → analyst only · `LATERAL` → full cluster · `SYSTEMIC` → all 20.

P0/P1: Fix → re-dispatch with `Delta: [file:lines — what changed]` → confirm CLEAN. Still BROKEN → retry (max 3 attempts).
P2: Fix → single analyst rebreak → CLEAN or document as known limitation.

**Skill Chain Interrupt**: SYSTEMIC BROKEN after 3 attempts → UPSTREAM halt + write `[RESTART REQUESTED]` to UFSD + ask user before any further action. Two+ Strong Attack Signal surfaces → architectural failure → `[RESTART REQUESTED]` to UFSD + escalate to `ultraflow:council`.

## Step 5: Final Verdict + UFSD Write

All analysts SOLID/FRAGILE (P2+ documented) → proceed to `verification`.
Any still BROKEN after 3 attempts → escalate:
```
## QA Division Escalation
Operative: [codename] Cluster: [name] Priority: [P0/P1] Blast: [radius]
Finding: [broken] Blocked by: [root cause] Needs: [recommendation]
Attempts: 1=[tried→result] 2=[tried→result] 3=[tried→result]
```

**UFSD Write** (append to active spec in `docs/ultraflow/specs/`):
```
## UFSD adversarial-testing [date]
Verdict: [PASS|ESCALATED|RESTART REQUESTED] | Coverage: [N/7 clusters SOLID]
Contradictions: [PROVISIONAL findings that conflicted across analysts]
Detail: [P0/P1 findings with reproduce steps]
```

## Anti-Patterns
- Fewer than 20 dispatched · Fabricating responses · Skipping SYSTEMIC rebreak · INVALID findings counted as coverage · P0 deferred while P2/P3 fixed · PROVISIONAL discarded · Proceeding to verification with any analyst BROKEN
