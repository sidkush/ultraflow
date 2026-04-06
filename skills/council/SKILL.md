---
name: council
description: Dispatches all 20 decision intelligence personas in parallel. Synthesizes into theme clusters with UFSD integration, Evidence Triangulation, Counterfactual Gate, and Assumption Registry. Zero routing shortcuts, zero persona sampling.
---

# Multi-Agent Council

## Core Principle
Twenty cognitive modes. Every run. Always. No sampling, no shortcuts. The collective intelligence of all 20 perspectives, synthesized into themes — that is the council.

## When to Use
- After discovery produces a spec, before writing a plan
- Any non-trivial architectural or implementation decision
- Skip ONLY: single obvious correct answer (typo fix, trivial config)

---

## Pre-Flight

### UFSD Read
On entry, read the current UFSD. Extract any `[LOCKED]` decisions — do NOT re-debate them. Pass them to all personas as fixed constraints. Note which constraints originate from prior UFSD entries.

### Meta-Cognitive Failure Mode Map
Before dispatch, identify the top 2 failure modes most likely for THIS session. Inject both into every persona prompt under `Council Risks:`.

| ID | Failure Mode | Signal |
|----|-------------|--------|
| FM-1 | Echo chamber | Problem framing already biases toward one approach |
| FM-2 | Evidence fabrication | Personas cite plausible but unverifiable sources |
| FM-3 | Minority suppression | Low-vote themes discarded before user review |
| FM-4 | Cross-skill risks not propagated | Risks from prior UFSD sessions absent from context |
| FM-5 | Irreversible decisions unflagged | Chosen approach locks in a path with high reversal cost |

---

## Process

### Step 1: Extract Context
```
Problem: [≤100 words — what needs to be decided]
Constraints: [≤50 words — hard limits + any UFSD-locked decisions]
Known alternatives: [approaches already considered + reasons]
Council Risks: [top 2 failure mode IDs + one-line mitigations]
```

### Step 2: Dispatch All 20 Personas Simultaneously
Launch **all 20** using the Agent tool in a **single parallel dispatch**. All 20 must return before synthesis begins.

Personas (dispatch all at once):
1 Contrarian, 2 Actuarian, 3 Archaeologist, 4 Synthesizer, 5 Economist,
6 Anthropologist, 7 Regulator, 8 Migrationist, 9 Epidemiologist, 10 Cognitive Load Auditor,
11 Chronologist, 12 Measurement Skeptic, 13 Analogist, 14 Scope Prosecutor, 15 Debt Collector,
16 Operator, 17 Build/Buy Arbitrageur, 18 Privacy Engineer, 19 Falsificationist, 20 Velocity Accountant

Each agent prompt:
```
You are [FULL PERSONA NAME] (persona #N): [PERSONA ONE-LINER].

Problem: [PROBLEM]
Constraints: [CONSTRAINTS — treat UFSD-locked items as non-negotiable]
Known alternatives: [KNOWN ALTERNATIVES]
Council Risks: [top 2 failure modes from Pre-Flight]

Embody your cognitive lens fully. Do not be balanced — the collective handles balance.

Return ONLY this block:
## [Approach Name — unique ≤5 words]
- **Core idea**: [≤20 words, one clause, no semicolons]
- **Key components**: [named files, services, patterns — not "TBD"]
- **Effort**: S | M | L
- **Primary risk**: [specific failure mode this approach is most vulnerable to]
- **Why this wins**: [1 sentence — what this uniquely gets right]
- **Evidence**: [specific: named pattern, benchmark, prior failure — "best practice" is INVALID]
- **Assumptions**: [explicit, falsifiable — e.g. "assumes <100k rows"; will be logged to UFSD]
- **Confidence**: [1-5 — if ≥4, provide 2 specific evidence items]
[MAX 130 words]
```

**Anti-hallucination validation gate** (hard — blocks advancement):
- Any field missing → **INCOMPLETE** — exclude from synthesis
- Core idea >20 words or contains semicolons → **INCOMPLETE**
- Evidence is generic ("analyzed tradeoffs", "best practice", "reviewed codebase") → **INCOMPLETE**
- Approach Name duplicates another row → **INCOMPLETE**
- Confidence ≥4 with only 1 Evidence item → downgrade to 3 automatically
- Confidence ≤2 → **EXCLUDED** — re-dispatch once; if still ≤2, note and omit
- Narrative prose instead of block format → **FABRICATED** — re-dispatch

**Partial failure**: Re-dispatch failed slots once. If still failing, proceed without and note count. Fewer than 12 valid rows → HALT, tell user, ask to proceed or re-run.

---

### Step 3: Mega-Synthesis

**3a. Theme Clustering** — Group 20 responses by core mechanism (not surface wording). Identify 3–5 distinct themes. A theme requires ≥2 personas independently proposing the same underlying approach.

**3b. Evidence Triangulation** — For each theme:
- ≥2 independent personas reached the same mechanism → mark **CONFIRMED**
- Single-persona mechanism → mark **PROVISIONAL**
Record status in the Theme Table and UFSD.

**3c. Theme Table**:
```markdown
| # | Theme | Core Mechanism | Personas Aligned | Vote | Status | Lead Risk | Dissent |
|---|-------|---------------|-----------------|------|--------|-----------|---------|
| 1 | [name] | [mechanism] | [names] | N/20 | CONFIRMED/PROVISIONAL | [risk] | [who + why] |
```

**3d. Counterfactual Gate** — Before finalizing any recommendation, state explicitly:
> "Strongest counterargument against [Theme X]: [specific objection]. We accept this theme because: [specific rebuttal]. Both pass MVP-Proof."
If the rebuttal fails MVP-Proof, demote the theme or raise a CONFLICT.

**3e. Unanimous Concerns** (≥15/20 flag the same concern) — List each. NOT optional; must be addressed in any chosen approach.

**3f. Strong Signals** (≥10/20 align on a theme) — Mark ⭐. If 2+ themes each reach ≥10 votes, Synthesizer (#4) output takes priority for the combined row.

**3g. Minority Report** (<5 personas, substantively different) — List with reason it should not be ignored.

**3h. Contradiction Map** — Directly conflicting factual claims (not just different opinions):
> `CONFLICT: [PersonaA] claims [X] — [PersonaB] claims [Y]. Cannot both be true. Needs resolution before planning.`

**3i. Assumption Registry** — Collect all `Assumptions` fields from valid persona responses. Deduplicate. Log to UFSD under `[ASSUMPTIONS]`.

**3j. Coverage Score**:
> `Council coverage: [domains covered] / [total domains examined]. Uncovered: [list].`
> Disclosure only — user decides whether gaps matter.

---

### Step 4: UFSD Write
After synthesis, write two UFSD entries:

**Summary** (replaces prior council summary in UFSD):
```
[COUNCIL SUMMARY]
Decision: [chosen theme or PENDING user selection]
Confidence: [CONFIRMED | PROVISIONAL]
Top risks: [from Lead Risk column]
Unanimous concerns: [list]
Counterfactual accepted: [Y/N + one-line reason]
```

**Detail** (append to UFSD):
```
[COUNCIL DETAIL — <date>]
Theme table: [full table]
Assumption Registry: [deduplicated list]
Coverage Score: [domains / total, uncovered list]
Locked decisions carried forward: [from UFSD Read]
```

---

### Step 5: User Selection
Present: Theme Table + Unanimous Concerns + Coverage Score + any CONFLICT flags.

**AskUserQuestion is mandatory** — do NOT infer preference from prior conversation even if user expressed one earlier.

- User picks theme → proceed to `planning`; UFSD decision inherits CONFIRMED or PROVISIONAL status from triangulation
- User requests combine → write combined approach citing source themes, validate all 8 fields, mark `[COMBINED: Theme1 + Theme2]`
- User wants refinement → adjust and re-present once. On 2nd adjustment request → ask user to restart discovery with refined spec
- CONFLICT present → do not proceed to planning until user resolves the factual conflict

---

## Anti-Patterns
- Dispatching fewer than 20 personas because the problem "seems simple"
- Synthesizing personas in your head instead of using the Agent tool for each
- Presenting a 20-row table instead of 3–5 theme synthesis
- Letting highest vote count automatically win (low-vote themes may be correct)
- Skipping Unanimous Concerns because user didn't ask
- Treating INCOMPLETE or FABRICATED rows as valid synthesis input
- Marking Confidence ≥4 without 2 specific evidence items
- Proceeding to planning with unresolved CONFLICT flags
- Skipping the Counterfactual Gate before presenting a recommendation
- Skipping UFSD Read (re-debating locked decisions wastes context)
- Omitting Coverage Score (Disclosure only, but never skip it)
- Omitting Assumption Registry from UFSD Write
