---
name: parallel-dispatch
description: Use when facing 2+ independent tasks without shared state or sequential dependencies. Dispatches focused agents in parallel, reviews and integrates results.
---

# Parallel Agent Dispatch

## When to Use
- 2+ tasks that don't depend on each other
- Tasks that read different files or produce different outputs
- Council persona dispatch (handled by council skill, but uses same pattern)
- Adversarial breaker dispatch (handled by adversarial-testing, same pattern)

## Process

### Step 1: Identify Independent Tasks
Tasks are independent when:
- They modify different files
- They don't read each other's output
- Order doesn't matter

Tasks are NOT independent when:
- Task B reads a file Task A creates
- Task B calls a function Task A defines
- They modify the same file

### Step 2: Create Agent Prompts
Each prompt must be:
- **Self-contained**: Include all context inline. Do NOT reference files the agent hasn't been given.
- **Focused**: One task per agent. No "also do X while you're at it."
- **Explicit about deliverables**: State exactly what output format you expect.
- **Under 300 words**: Keep prompts tight to save tokens.
- **Require confidence score**: Every agent must end with `Confidence: [1-5]`. Score 1-2 = re-dispatch with more context. Score 3 = review carefully. Score 4-5 = trust but verify.

### Step 3: Dispatch
Use the Agent tool with multiple calls in a single message. All independent agents launch simultaneously.

### Step 4: Review and Integrate
After all agents return:
1. Read each agent's output
2. Check for conflicts (did two agents modify the same concept differently?)
3. If conflicts → resolve manually, don't re-dispatch
4. Run full test suite to verify integration
5. Commit integrated changes

## Token Budget
- ~300 tokens per agent prompt
- Keep to 2-5 agents per dispatch (more = diminishing returns + conflict risk)

## Anti-Patterns
- Dispatching dependent tasks in parallel (causes conflicts)
- Agent prompts that say "read the plan file" (provide content inline)
- More than 5 parallel agents (too many conflicts to resolve)
- Not running tests after integration
