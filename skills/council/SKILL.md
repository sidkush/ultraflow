---
name: council
description: Always dispatches all 20 decision intelligence personas in parallel. Synthesizes into theme clusters with vote counts. Zero routing shortcuts, zero persona sampling. The collective is the product.
---

# Multi-Agent Council

## Core Principle
Twenty cognitive modes. Every run. Always. No sampling, no routing shortcuts, no "5 is enough." The collective intelligence of all 20 perspectives, synthesized into themes — that is the council.

## When to Use
- After discovery produces a spec, before writing a plan
- When facing any non-trivial architectural or implementation decision
- When the user explicitly asks for options or alternatives
- Skip ONLY for tasks with a single obvious correct answer (typo fix, clear spec, trivial config)

## Process

### Step 1: Extract Context
Before dispatch, distill from the problem:
```
Problem: [≤100 words — what needs to be decided]
Constraints: [≤50 words — hard limits: tech stack, timeline, team size, non-negotiables]
Known alternatives: [list any approaches already considered, with reasons they were considered]
```

### Step 2: Dispatch All 20 Personas Simultaneously
Launch **all 20 decision personas** using the Agent tool — in a single parallel dispatch. All 20 must return before synthesis begins.

The 20 personas (dispatch in this order, all at once):
1 Contrarian, 2 Actuarian, 3 Archaeologist, 4 Synthesizer, 5 Economist,
6 Anthropologist, 7 Regulator, 8 Migrationist, 9 Epidemiologist, 10 Cognitive Load Auditor,
11 Chronologist, 12 Measurement Skeptic, 13 Analogist, 14 Scope Prosecutor, 15 Debt Collector,
16 Operator, 17 Build/Buy Arbitrageur, 18 Privacy Engineer, 19 Falsificationist, 20 Velocity Accountant

Each agent prompt:
```
You are [FULL PERSONA NAME] (persona #N): [PERSONA ONE-LINER from personas.md].

Problem: [PROBLEM from Step 1]
Constraints: [CONSTRAINTS from Step 1]
Known alternatives: [KNOWN ALTERNATIVES from Step 1]

Your job: propose the single best approach FROM YOUR COGNITIVE LENS. Do not try to be balanced — embody your perspective fully. The collective will handle balance.

Return ONLY this block:
## [Approach Name — unique ≤5 words]
- **Core idea**: [≤20 words, one clause, no semicolons — the mechanism, not the goal]
- **Key components**: [specific named files, services, patterns — not "TBD" or "depends"]
- **Effort**: S | M | L
- **Primary risk**: [specific failure mode this approach is most vulnerable to]
- **Why this wins**: [1 sentence — what this approach uniquely gets right]
- **Evidence**: [specific: a named pattern, benchmark, prior failure, or constraint — "reviewed codebase" is INVALID]
- **Assumptions**: [what must be true for this to work — explicit, falsifiable, e.g. "assumes <100k rows"]
- **Confidence**: [1-5 — if ≥4, provide 2 specific evidence items]
[MAX 130 words]
```

**Anti-hallucination validation gate** (hard — blocks advancement if violated):
- Any field missing → **INCOMPLETE** — do not include in synthesis
- Core idea >20 words or contains semicolons → **INCOMPLETE**
- Evidence is generic ("analyzed tradeoffs", "best practice", "reviewed codebase") → **INCOMPLETE**
- Approach Name duplicates another row → **INCOMPLETE** (must be unique)
- Confidence ≥4 with only 1 Evidence item → downgrade Confidence to 3 automatically
- Confidence ≤2 → **EXCLUDED** — re-dispatch once; if still ≤2, note to user and omit
- Any agent that returns narrative prose instead of the block format → **FABRICATED** — treat as if not dispatched, re-dispatch

**Partial failure**: If any agents fail/error → re-dispatch those slots once. If re-dispatch also fails → proceed without them, note count. If fewer than 12 valid rows remain → HALT, tell user how many succeeded, ask whether to proceed or re-run.

### Step 3: Mega-Synthesis — Themes, Not Rows
After all 20 agents return, do NOT present 20 individual rows. Instead:

**3a. Theme Clustering**: Group the 20 responses by core mechanism (not surface wording). Identify 3–5 distinct themes. A theme is a group of ≥2 personas independently proposing the same underlying approach.

**3b. Theme Table**:
```markdown
| # | Theme | Core Mechanism | Personas Aligned | Vote | Lead Risk | Dissent |
|---|-------|---------------|-----------------|------|-----------|---------|
| 1 | [name] | [mechanism] | [names] | N/20 | [risk] | [who disagrees + why] |
...
```

**3c. Unanimous Concerns** (≥15/20 personas flag the same concern):
→ List each. These are NOT optional — they must be addressed in any approach chosen.

**3d. Strong Signals** (≥10/20 personas align on a theme):
→ Mark with ⭐. If 2+ themes each have ≥10 votes, the Synthesizer's output (persona #4) takes priority for the combined row.

**3e. Minority Report** (raised by <5 personas but substantively different):
→ List with reason it shouldn't be ignored. Do not discard minority views.

**3f. Contradiction Map**: If persona A and persona B make directly conflicting factual claims (not just different opinions):
→ Flag as: `CONFLICT: [PersonaA] claims [X] — [PersonaB] claims [Y]. Cannot both be true. Needs resolution before planning.`

### Step 4: User Selection
Present the Theme Table (3–5 rows, not 20) + Unanimous Concerns + any CONFLICT flags.

**AskUserQuestion is mandatory** — do NOT infer preference from prior conversation context even if user expressed one earlier.

- User picks theme → proceed to `planning` with that theme as the approach
- User requests combine → write combined approach citing source themes, validate all 8 fields, mark `[COMBINED: Theme1 + Theme2]`
- User wants refinement → adjust and re-present once. On 2nd adjustment request → ask user to restart discovery with refined spec
- CONFLICT present → do not proceed to planning until user resolves the factual conflict

## Anti-Patterns
- Dispatching fewer than 20 personas because the problem "seems simple"
- Synthesizing personas in your head instead of using the Agent tool for each
- Presenting a 20-row table instead of a 3–5 theme synthesis
- Letting one theme "win" just because it has the most votes (low-vote themes may be correct)
- Skipping Unanimous Concerns because the user didn't ask about them
- Treating INCOMPLETE or FABRICATED rows as valid input to synthesis
- Marking Confidence ≥4 without 2 specific evidence items
- Proceeding to planning with unresolved CONFLICT flags
