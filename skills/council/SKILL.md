---
name: council
description: Use when choosing between implementation approaches. Dispatches 3–5 diverse thinkers in parallel, validates real dispatch, synthesizes top options for user selection.
---

# Multi-Agent Council

## Core Principle
One perspective produces one solution. Twenty cognitive modes produce the best solution. Dispatch diverse thinkers, validate real dispatch, synthesize their output.

## When to Use
- After discovery produces a spec, before writing a plan
- When facing a non-trivial architectural decision
- When the user explicitly asks for options or alternatives
- Skip for trivial tasks where the approach is obvious

## Process

### Step 1: Select Personas (3–5)
Read `personas.md`. Personas 1–20 are decision-intelligence thinkers. Personas 21–30 are specialist investigators.
**Dispatch 5** for complex/architectural decisions (≥3 real tradeoffs). **Dispatch 3** for decisions with exactly 1–2 real tradeoffs within a single domain. When in doubt, dispatch 5.

**Decision personas (1–20) — planning, architecture, implementation:**
| Domain | Select These 5 |
|--------|----------------|
| Frontend/UI | Anthropologist, Cognitive Load Auditor, Operator, Scope Prosecutor, Contrarian |
| Backend/API | Epidemiologist, Operator, Falsificationist, Chronologist, Actuarian |
| Data/Pipeline | Measurement Skeptic, Privacy Engineer, Epidemiologist, Actuarian, Falsificationist |
| DevOps/Infra | Operator, Migrationist, Actuarian, Debt Collector, Build/Buy Arbitrageur |
| Auth/Security | Regulator, Privacy Engineer, Actuarian, Falsificationist, Contrarian |
| Architecture | Archaeologist, Contrarian, Chronologist, Synthesizer, Scope Prosecutor |
| Full-stack/Complex | Scope Prosecutor, Synthesizer, Chronologist, Economist, Epidemiologist |
| New Feature | Anthropologist, Build/Buy Arbitrageur, Velocity Accountant, Scope Prosecutor, Contrarian |
| Refactor | Debt Collector, Migrationist, Operator, Archaeologist, Actuarian |
| Payments/Financial | Regulator, Actuarian, Privacy Engineer, Epidemiologist, Debt Collector |
| Algorithm/Correctness | Falsificationist, Actuarian, Measurement Skeptic, Analogist, Archaeologist |
| Creative/Experimental | Analogist, Contrarian, Scope Prosecutor, Falsificationist, Synthesizer |

**Cross-domain problems**: Take the union of applicable rows, remove duplicates, keep first 5 by priority order: Auth/Security > Data > Backend > rest.

**Specialist personas (21–30) — debugging, investigation, correctness:**
| Domain | Select These 5 |
|--------|----------------|
| Bug investigation | Forensic Pathologist, Bloodhound, Cartesian, Ghost Hunter, Binary Monk |
| Security/adversarial | Paranoid Security Auditor, Pessimistic Realist, Mad Scientist, Architect of Ruin, Forensic Pathologist |
| Algorithm/correctness | Mathematical Formalist, Cartesian, Falsificationist (persona 19), Bloodhound, Archaeologist (persona 3) |
| Distributed/systems bugs | Architect of Ruin, Ghost Hunter, Pessimistic Realist, Forensic Pathologist, Epidemiologist (persona 9) |

### Step 2: Dispatch Parallel Agents
Launch 3–5 agents **simultaneously using the Agent tool**. Each agent's response is a returned tool result — inline synthesis is fabrication and is a protocol violation.

Each agent gets:
```
You are [FULL PERSONA NAME] (persona #N in council): [PERSONA ONE-LINER].
Problem: [SPEC_SUMMARY — max 100 words]
Constraints: [KEY_CONSTRAINTS — max 50 words]

Return ONLY this block — no prose outside it:
## [Approach Name — unique, max 5 words]
- **Core idea**: [≤20 words — ONE clause, no semicolons]
- **Key files/components**: [specific names — not "TBD" or "depends on stack"]
- **Estimated effort**: S | M | L
- **Biggest risk**: [1 sentence — specific failure mode]
- **Why this works**: [1 sentence — mechanism, not restatement of core idea]
- **Evidence**: [specific constraint, metric, pattern name, or tradeoff — "reviewed codebase" is invalid]
- **Assumptions**: [what must be true for this to work — explicit, not implied]
- **Confidence**: [1-5]
[MAX 130 words]
```

**Validation gate** (block advancement if violated):
- Missing any of the 8 required fields → **INCOMPLETE**
- Core idea >20 words or contains semicolons → **INCOMPLETE**
- Evidence is generic ("analyzed tradeoffs", "reviewed codebase") → **INCOMPLETE**
- Approach Name duplicates an existing row → **INCOMPLETE** (rename or merge)
- Confidence ≤2 → re-dispatch that slot once; if still ≤2 → **EXCLUDED** (note to user)
- Fewer than 3 genuinely distinct Core ideas (distinct = different primary mechanism, not just different wording) → flag low-diversity, ask user before synthesizing

**Partial failure rules:**
- 1 agent errors → mark **FAILED**, re-dispatch that slot once
- Re-dispatched slot still fails → proceed without it (if ≥3 valid rows remain)
- Fewer than 3 valid rows after all re-dispatches → **HALT**, escalate to user — do not synthesize below 3

### Step 3: Synthesize
After all agents return, create the synthesis table:

```markdown
| # | Approach | Core Idea | Effort | Risk | From | Confidence |
|---|----------|-----------|--------|------|------|------------|
| 1 | [name]   | [≤10 words] | S/M/L | [risk] | [Persona #N] | [1-5] |
...
| C | [Combined: Source1 + Source2] | [what it takes from each] | ... | [Combined] | — |
```

Rules:
- At least 2 rows must be **pure** (single persona, unmodified)
- At least 1 row must be **[Combined: PersonaA + PersonaB]** — must name both sources and state what each contributes
- Rank by: feasibility first, then quality, then innovation
- If ≥3 personas independently converge on the same approach → call out as **strong signal**
- Rows marked INCOMPLETE, FAILED, or EXCLUDED → omit from ranking (note count to user)

### Step 4: User Selection
Present with AskUserQuestion. **Do NOT infer user preference from prior conversation context — AskUserQuestion is mandatory even if user expressed a preference earlier.**

- User picks option → proceed to `planning` with the chosen approach
- User requests combine → write combined approach, apply all 8 validation fields, mark as `[COMBINED: source1 + source2]`, proceed to `planning`. Do not re-dispatch agents.
- User wants changes → adjust table and re-present (max 2 adjustment rounds; on 3rd request, ask user to clarify requirements or re-run discovery first)

## Anti-Patterns
- Dispatching agents then synthesizing in your own reasoning instead of using returned results
- Making all 5 options variations of the same mechanism with different labels
- Letting the Contrarian (persona #1) dominate — dissent is a signal, not a veto
- Routing to Architect by default when no domain matches (use Scope Prosecutor + Analogist instead)
- Skipping Step 1 and hard-coding Architect/Pragmatist in every dispatch
- Proceeding to synthesis with fewer than 3 valid rows
