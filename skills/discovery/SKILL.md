---
name: discovery
description: Use before any creative or implementation work. Progressive requirement discovery — helps users who can't yet visualize their needs find clarity through staged exploration with option cards and confidence checks.
---

# Progressive Requirement Discovery

## Core Principle
Users often don't know what they want until they see options. Don't ask "what do you want?" — show them possibilities and let them react. Each stage sharpens their vision.

## When to Use
- Before ANY feature, refactor, or creative work
- When requirements are vague, incomplete, or aspirational
- When the user says "I want something like..." or "Can you build..."
- Skip ONLY for: typo fixes, explicit single-line changes, or tasks with a complete spec already provided

## The 5 Stages

### Stage 1: Domain Scan (~2 questions)
Ask exactly 2 questions using AskUserQuestion:
1. **What area?** (e.g., "frontend component", "API endpoint", "data pipeline", "CLI tool")
2. **What scale?** (e.g., "quick utility", "feature addition", "new system", "full application")

Do NOT proceed without answers. These two answers determine which option cards to generate.

### Stage 2: Option Cards (present 3 approaches)
Based on Stage 1, generate **exactly 3** genuinely different approaches as a table: `| # | Approach | Description | Pros | Cons | Effort |`. One simple, one thorough, one creative. Present with AskUserQuestion: "Which approach? Pick, combine, or describe different."

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

## Transition
When Stage 5 passes (4+):
- Save spec to `docs/ultraflow/specs/YYYY-MM-DD-<feature>.md`
- If implementation approach needs evaluation → invoke `council`
- If approach is clear → invoke `planning`

## Minimum Floor (Non-Negotiable)
Even when adapting depth for simple tasks, you MUST always complete:
- Stage 1 (Domain Scan) — always
- Stage 2 (Option Cards) — always
Stages 3-5 may be compressed for straightforward tasks but never skipped for anything beyond a single-function change.

## Anti-Patterns
- Asking 10 questions upfront (overwhelming, user disengages)
- Accepting "just build something" without at least Stage 1-2
- Generating options that are too similar
- Skipping confidence check (user may silently disagree)
- Writing a full design doc when bullet points suffice
- Using token budget as justification to skip Stages 1-2
