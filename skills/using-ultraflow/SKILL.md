---
name: using-ultraflow
description: Use at conversation start. Establishes skill routing, token-aware workflow, and instruction priority. Invoke BEFORE any response including clarifying questions.
---

## Instruction Priority
1. **User instructions** (CLAUDE.md, direct requests) — highest
2. **UltraFlow skills** — override defaults where they conflict
3. **System prompt** — lowest

## The Rule
**Invoke relevant skills BEFORE any response or action.** Even 1% chance = invoke it.

## Intent Echo (Mandatory)
Before executing ANY user request, echo back what you understood in 1 sentence. Wait for user confirmation. This catches misinterpretation before work begins. Format: "I understand you want me to: [1 sentence]. Correct?"

If user says **No**: ask "What should I correct?" Re-echo once with the correction applied. If still rejected → proceed with best interpretation and note inline: "(Proceeding with best understanding — please correct mid-task if needed.)"

## Available Skills

| Skill | Trigger | ~Tokens |
|-------|---------|---------|
| `discovery` | Before any creative/implementation work | 800 |
| `council` | When choosing between implementation approaches | 300+1500 |
| `planning` | When spec exists, before coding | 600 |
| `building` | When plan exists, ready to implement | 700 |
| `debugging` | Any bug, test failure, unexpected behavior | 500 (simple) / 1500 (complex w/ Skeptic) |
| `adversarial-testing` | After implementation, before declaring done | 400 |
| `verification` | Before claiming work is complete | 300 |
| `git-flow` | Starting isolated work or finishing a branch | 400 |
| `receiving-review` | When receiving code review feedback | 250 |
| `parallel-dispatch` | 2+ independent tasks without shared state | 300 |

## Workflow Graph
```
discovery → council → planning → building → adversarial-testing → verification → git-flow
                                     ↕              ↕                                ↓
                                 debugging    parallel-dispatch          receiving-review (on PR feedback)
```

## Token Budget Awareness
Each skill lists its token cost. Before invoking expensive skills, consider:
- Is the task complex enough to justify the cost?
- Can a lighter skill handle it?
- For trivial tasks (typo, single-line change): skip discovery/council, go straight to building.
- **Token budget NEVER justifies skipping rigid skills** (building, debugging, verification).

## Anti-Drift Protocol (Context Decay Detection)
Track tasks since last skill re-read. Thresholds:
- **Every 5 tasks**: Re-read this file silently to recalibrate.
- **Every 10 tasks**: Re-read ALL active skill files.
- **After any context compaction**: Re-invoke this skill immediately.
- **Never trust a subagent's claim** without running verification yourself.
If you notice yourself skipping steps or abbreviating processes → you are drifting. Re-read NOW.

## Skill Types
- **Rigid** (building, debugging, verification, receiving-review): Follow exactly. No shortcuts. Note: debugging has a Simple/Complex tiered entry — Simple path is an explicit branch, not a shortcut.
- **Flexible** (discovery, council, planning): Adapt depth to task complexity.
  - **Minimum floor for flexible skills**: discovery must always complete Stages 1-2. Council must dispatch at least 3 real agents. Planning must always run self-review.

## Red Flags
Any thought starting with "this doesn't need a skill", "let me just do X first", "the skill is overkill", "I need context first", or "it's just a simple question" is rationalization. **Check the skill table before acting.**
