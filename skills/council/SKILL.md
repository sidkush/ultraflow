---
name: council
description: Use when choosing between implementation approaches. Dispatches 5 persona-driven agents in parallel to generate diverse solutions, then synthesizes top 5 options for user selection.
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

### Step 1: Select 5 Personas
Read `personas.md` (in this skill's directory). Choose 5 of 10 based on problem domain:

- **Frontend/UI** → Pragmatist, User Advocate, Minimalist, Architect, Optimizer
- **Backend/API** → Architect, Guardian, Optimizer, Researcher, Pragmatist
- **Data/Pipeline** → Researcher, Optimizer, Guardian, Architect, Minimalist
- **DevOps/Infra** → Guardian, Pragmatist, Optimizer, Integrator, Architect
- **Full-stack/Complex** → Architect, Pragmatist, Guardian, User Advocate, Researcher
- **Creative/Experimental** → Hacker, Futurist, User Advocate, Researcher, Minimalist

### Step 2: Dispatch Parallel Agents
Launch 5 agents simultaneously using the Agent tool. Each gets:

```
You are [PERSONA_NAME]: [PERSONA_ONE_LINER].
Problem: [SPEC_SUMMARY — max 100 words]
Constraints: [KEY_CONSTRAINTS — max 50 words]

Propose ONE implementation approach in this format:
## [Approach Name]
- **Core idea**: [1 sentence]
- **Key files/components**: [list]
- **Estimated effort**: [S/M/L]
- **Biggest risk**: [1 sentence]
- **Why this works**: [1 sentence]

Keep your response under 150 words total.
```

IMPORTANT: Each agent prompt must be self-contained. Do NOT reference files or context outside the prompt.

### Step 3: Synthesize
After all 5 return, create a synthesis table:

```markdown
| # | Name | Core Idea | Effort | Risk | From |
|---|------|-----------|--------|------|------|
| 1 | ...  | ...       | S/M/L  | ...  | [Persona] |
| 2 | ...  | ...       | S/M/L  | ...  | [Persona] |
| 3 | ...  | ...       | S/M/L  | ...  | [Combined] |
| 4 | ...  | ...       | S/M/L  | ...  | [Persona] |
| 5 | ...  | ...       | S/M/L  | ...  | [Combined] |
```

Rules:
- At least 2 options should be pure (from single persona)
- At least 1 should combine strengths from multiple personas
- Rank by: feasibility first, then quality, then innovation
- If personas agree on an approach, that's a strong signal — highlight it

### Step 4: User Selection
Present with AskUserQuestion: "Which approach should we plan? Pick one, combine, or refine."

If user picks → proceed to `planning` with the chosen approach.
If user wants changes → adjust and re-present (no need to re-run council).

## Token Budget
- Skill load: ~300 tokens
- 5 persona dispatches: ~1500 tokens (300 each including response)
- Synthesis: ~500 tokens
- Total: ~2300 tokens

## Minimum Floor (Non-Negotiable)
Even when adapting for simpler decisions, you MUST dispatch at least **3 real agents** using the Agent tool. Synthesizing perspectives in your head without actual dispatch is NOT the council — it's guessing.

## Anti-Patterns
- Running council for trivial decisions (use judgment)
- Making all 5 options variations of the same idea
- Ignoring minority perspectives that raise valid concerns
- Re-running council when the user just wants a small tweak
- Fabricating agent responses instead of actually dispatching agents
