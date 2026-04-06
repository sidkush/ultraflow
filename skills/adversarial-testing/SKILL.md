---
name: adversarial-testing
description: Dispatches ALL 20 NEMESIS operatives in parallel to find bugs, security holes, regressions, and edge cases. Zero sampling. Full division every run.
---

# Adversarial Testing — NEMESIS Division

> 20 operatives. Every run. Full strength or not at all.

## Meta-Cognitive Failure Mode Map (check before Step 1)
1. **Scope drift** — operatives attack adjacent systems. Fix: lock attack surface in Step 1.
2. **Hallucinated findings** — fabricated reproduce steps. Fix: anti-hallucination gate (Step 2).
3. **UFSD blind spot** — council-flagged risks not used as attack vectors. Fix: UFSD Read (Step 1).
4. **Confirmation bias** — operatives validate instead of break. Fix: enforce BROKEN|FRAGILE|SOLID only.
5. **Coverage illusion** — INVALID findings counted as coverage. Fix: Coverage Score disclosure (Step 3).

Top 2 failure modes per operative are in `breaker-personas.md`.

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

Operative prompt template:
```
You are [CODENAME]: [MANTRA]. Specialty: [ATTACK_SPECIALTY]. Moves: [4 SIGNATURE_MOVES].
Target: [TARGET] | Surface: [ATTACK_SURFACE] | Code: [RELEVANT_CODE ≤60 lines]
Break this. Return ONLY:
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
- Critical finding, no label → auto **high**

**Partial failure**: Re-dispatch failed slots once. < 12 valid reports → HALT, tell user count, ask to proceed.

**Evidence Triangulation**: VALID = ≥2 operatives independently agree OR 1 operative with 2 distinct proof paths. Single-operative, single-path = **PROVISIONAL**. Mark in triage; do not discard.

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
- **Minority Discoveries** (<3 operatives, critical/high) → list with name; never discard.
- **Clean Operatives** → list SOLID reporters (confirmed-safe surfaces).

**Coverage Score**: `Attack surface: [N clusters SOLID] / 7.` Disclosure only — user decides if sufficient.

## Step 4: Fix-and-Rebreak Cycle

Rebreak scope: `CONTAINED` → operative only · `LATERAL` → full cluster · `SYSTEMIC` → all 20.

P0/P1: Fix → re-dispatch with `Delta: [file:lines — what changed]` → confirm CLEAN. Still BROKEN → retry (max 3 attempts).
P2: Fix → single operative rebreak → CLEAN or document as known limitation.

**Skill Chain Interrupt**: SYSTEMIC BROKEN after 3 attempts → write `[RESTART REQUESTED]` to UFSD + ask user before any further action. Two+ Strong Attack Signal surfaces → architectural failure → `[RESTART REQUESTED]` to UFSD + escalate to `ultraflow:council`.

## Step 5: Final Verdict + UFSD Write

All operatives SOLID/FRAGILE (P2+ documented) → proceed to `verification`.
Any still BROKEN after 3 attempts → escalate:
```
## NEMESIS Escalation
Operative: [codename] Cluster: [name] Priority: [P0/P1] Blast: [radius]
Finding: [broken] Blocked by: [root cause] Needs: [recommendation]
Attempts: 1=[tried→result] 2=[tried→result] 3=[tried→result]
```

**UFSD Write** (append to active spec in `docs/ultraflow/specs/`):
```
## UFSD adversarial-testing [date]
Verdict: [PASS|ESCALATED|RESTART REQUESTED] | Coverage: [N/7 clusters SOLID]
Contradictions: [PROVISIONAL findings that conflicted across operatives]
Detail: [P0/P1 findings with reproduce steps]
```

## Anti-Patterns
- Fewer than 20 dispatched · Fabricating responses · Skipping SYSTEMIC rebreak · INVALID findings counted as coverage · P0 deferred while P2/P3 fixed · PROVISIONAL discarded · Proceeding to verification with any operative BROKEN
