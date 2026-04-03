---
name: discovery
description: Use before any creative or implementation work. Progressive requirement discovery — helps users who can't yet visualize their needs find clarity through staged exploration with option cards and confidence checks.
---

# Progressive Requirement Discovery

## Core Principle
Users often don't know what they want until they see options. Don't ask "what do you want?" — show them possibilities and let them react. Each stage sharpens their vision.

**Skip discovery entirely for**: typo fixes, single-line changes where the change is completely explicit, or tasks where the user has already provided a complete spec.

## The 5 Stages

### Stage 1: Domain Scan (~2 questions)
Ask exactly 2 questions using AskUserQuestion:
1. **What area?** (e.g., "frontend component", "API endpoint", "data pipeline", "CLI tool")
2. **What scale?** (e.g., "quick utility", "feature addition", "new system", "full application")

Do NOT proceed without answers. These two answers determine which option cards to generate.

### Stage 2: Option Cards (present 3 approaches)
Based on Stage 1, generate **exactly 3** genuinely different approaches as a table: `| # | Approach | Description | Pros | Cons | Effort |`. One simple, one thorough, one creative. **Each option must differ in its primary trade-off** — not just implementation style. If two options would make the same fundamental bet, merge them and generate a genuinely different third. If all three make the same fundamental bet → discard all and generate three new options. If the second set is also all-identical → stop and ask user: "No distinct approaches found. What trade-offs matter most to you?" Use the answer to generate options. Present with AskUserQuestion: "Which approach? Pick, combine, or describe different."

### Stage 3: Deep Dive (3-5 targeted questions)
Based on chosen approach, ask 3-5 focused questions about:
- **Constraints**: What must it NOT do? Existing patterns to follow?
- **Edge cases**: What happens when X fails? Empty states?
- **Integration**: What does it connect to? Data sources? APIs?
- **Users**: Who interacts with this? What's their skill level?

Ask questions ONE AT A TIME using AskUserQuestion. Each question should build on previous answers. Stop when you have enough to write a spec.

### Stage 4: Spec Preview
Present a mini-spec as bullet points: What We're Building (1-line), Key Decisions (3-5 bullets), Scope IN/OUT, Success Criteria (2-3 measurable). Ask: "Does this capture your vision?"

### Stage 5: Confidence Check
Ask using AskUserQuestion: "On a scale of 1-5, how clear is your vision now?"
- **5**: Perfect, proceed to planning/council
- **4**: Good enough, proceed with noted uncertainties
- **3**: Revisit Stage 3 — ask 2 more targeted questions, then re-present spec
- **1-2**: Revisit Stage 2 — the chosen approach may not be right. Generate 3 new options informed by what you've learned

**Iteration cap**: If confidence remains below 4 after **2 revisit cycles** → stop looping. Present the current spec and ask: "What specific part is still unclear?" Incorporate the answer as an open question in the spec, prefix the spec filename with `DRAFT-`, and add `⚠️ Incomplete — uncertainty: [topic]` to the spec header. Proceed to planning with this flag visible so downstream skills treat it as provisional.

## Transition
When Stage 5 passes (4+):
- Save spec to `docs/ultraflow/specs/YYYY-MM-DD-<feature>.md` (or `DRAFT-YYYY-MM-DD-<feature>.md` if iteration cap fired)
- If implementation approach needs evaluation → invoke `council`
- If approach is clear → invoke `planning`

## Minimum Floor
Always complete Stage 1 + Stage 2. Stages 3-5 may be compressed but not skipped for anything beyond a single-function change. **Skip condition takes priority**: if the user provides a complete spec, skip discovery entirely regardless of task size.

## Anti-Patterns
- Asking 10 questions upfront (overwhelming, user disengages)
- Accepting "just build something" without at least Stage 1-2
- Generating options that are too similar
- Skipping confidence check (user may silently disagree)
- Writing a full design doc when bullet points suffice
- Using token budget as justification to skip Stages 1-2
