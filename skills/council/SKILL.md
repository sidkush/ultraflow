---
name: council
description: Use when choosing between implementation approaches. Dispatches 3–5 persona-driven agents in parallel to generate diverse solutions, then synthesizes top options for user selection.
---

# Multi-Agent Council

## Core Principle
One perspective produces one solution. Five perspectives produce the best solution. Dispatch diverse thinkers, synthesize their output.

## When to Use
- After discovery produces a spec, before writing a plan
- When facing a non-trivial architectural decision
- When the user explicitly asks for options or alternatives
- Skip for trivial tasks where the approach is obvious

## Process

### Step 1: Select Personas (3–5)
Read `personas.md` (in this skill's directory). Choose based on problem domain. **Dispatch 5** for complex/architectural decisions. **Dispatch 3** for decisions with ≤2 real tradeoffs.

**Generalist personas (1–10) — use for planning, architecture, implementation decisions:**
- **Frontend/UI** → Pragmatist, User Advocate, Minimalist, Architect, Optimizer
- **Backend/API** → Architect, Guardian, Optimizer, Researcher, Pragmatist
- **Data/Pipeline** → Researcher, Optimizer, Guardian, Architect, Minimalist
- **DevOps/Infra** → Guardian, Pragmatist, Optimizer, Integrator, Architect
- **Full-stack/Complex** → Architect, Pragmatist, Guardian, User Advocate, Researcher
- **Creative/Experimental** → Hacker, Futurist, User Advocate, Researcher, Minimalist

**Specialist personas (11–20) — use for investigation, debugging, adversarial, correctness:**
- **Bug investigation** → Forensic Pathologist, Bloodhound, Cartesian, Ghost Hunter, Binary Monk
- **Security/adversarial** → Paranoid Security Auditor, Guardian, Pessimistic Realist, Mad Scientist, Architect of Ruin
- **Algorithm/correctness** → Mathematical Formalist, Cartesian, Researcher, Architect, Bloodhound
- **Distributed/systems bugs** → Architect of Ruin, Ghost Hunter, Pessimistic Realist, Forensic Pathologist, Integrator

### Step 2: Dispatch Parallel Agents
Launch 3–5 agents simultaneously using the Agent tool. Each gets:

```
You are [PERSONA_NAME]: [PERSONA_ONE_LINER].
Problem: [SPEC_SUMMARY — max 100 words]
Constraints: [KEY_CONSTRAINTS — max 50 words]

Return ONLY this block — no prose outside it:
## [Approach Name]
- **Core idea**: [1 sentence]
- **Key files/components**: [list]
- **Estimated effort**: S | M | L
- **Biggest risk**: [1 sentence]
- **Why this works**: [1 sentence]
- **Evidence**: [what you analyzed — specific tradeoff, constraint, or pattern — min 1 item]
- **Confidence**: [1-5]
[MAX 120 words]
```

IMPORTANT: Each agent prompt must be self-contained. Do NOT reference files or context outside the prompt.

**Partial failure handling**: If 1 agent returns nothing or errors → mark that row FAILED in synthesis, proceed with remaining 4. If 2+ agents fail → re-dispatch failed agents once before synthesizing. If re-dispatch also fails → proceed with available results and note incomplete coverage to user.

**Validation gate** (before synthesizing): Verify each response has all 7 required fields. Missing any → mark row INCOMPLETE. Confidence ≤2 → treat as FAILED. If fewer than 3 genuinely distinct Core ideas exist across all valid rows → flag low-diversity and ask user: "These approaches are very similar. Should I re-dispatch with forced differentiation, or proceed?" On re-dispatch: assign each agent a different required trade-off axis (e.g., "prioritize speed", "prioritize safety", "minimize complexity").

### Step 3: Synthesize
After all agents return, create a synthesis table (one row per agent, plus combined rows if warranted):

```markdown
| # | Name | Core Idea | Effort | Risk | From | Confidence |
|---|------|-----------|--------|------|------|------------|
| 1 | ...  | ...       | S/M/L  | ...  | [Persona] | [1-5] |
| 2 | ...  | ...       | S/M/L  | ...  | [Persona/Combined] | [1-5] |
...
```

Rules:
- At least 2 options should be pure (from single persona)
- At least 1 should combine strengths from multiple personas
- Rank by: feasibility first, then quality, then innovation
- If personas agree on an approach, that's a strong signal — highlight it
- Exclude rows marked INCOMPLETE or FAILED from ranking unless all rows are incomplete

### Step 4: User Selection
Present with AskUserQuestion: "Which approach should we plan? Pick one, combine, or refine."

If user picks → proceed to `planning` with the chosen approach.
If user wants changes → adjust and re-present (no need to re-run council).

## Minimum Floor (Non-Negotiable)
**5 agents is the target.** For genuinely simpler decisions (only 1-2 real tradeoffs), 3 agents are acceptable. Fewer than 3 is never acceptable — that is guessing, not council. Synthesizing perspectives in your head without actual dispatch is NOT the council.

## Anti-Patterns
- Running council for trivial decisions (use judgment)
- Making all 5 options variations of the same idea
- Ignoring minority perspectives that raise valid concerns
- Re-running council when the user just wants a small tweak
- Fabricating agent responses instead of actually dispatching agents
