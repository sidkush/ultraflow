# UltraFlow: Detailed Explanation

## What UltraFlow Is

UltraFlow is an enhanced agentic skills framework for coding agents (Claude Code and compatible platforms). It is an improved, token-efficient alternative to the [Superpowers](https://github.com/obra/superpowers) framework, designed to produce better results while consuming approximately 55% fewer tokens.

---

## What Was Added Beyond Superpowers

### 1. Progressive Requirement Discovery (replaces Brainstorming)

**Superpowers approach**: Linear brainstorming — asks questions one at a time, proposes approaches, writes a design doc. Users must already know what they want.

**UltraFlow approach**: A 5-stage funnel designed for users who *can't yet visualize* their needs:

| Stage | Purpose | Why It Helps |
|-------|---------|--------------|
| Domain Scan | 2 quick questions to narrow the space | Prevents wasted exploration |
| Option Cards | Show 3 genuinely different approaches as a table | Users react to options better than they answer open questions |
| Deep Dive | 3-5 targeted questions on chosen approach | Builds on user's reaction rather than starting from scratch |
| Spec Preview | Bullet-point mini-spec for approval | Concrete enough to spot misalignment early |
| Confidence Check | "How clear is your vision? 1-5" | If below 4, loops back — never proceeds with unclear requirements |

**Why it performs better**: Users who start vague end up with crystal-clear specs because each stage builds on the previous. The option cards are the key innovation — showing three genuinely different interpretations helps users discover what they actually want by reaction rather than creation.

### 2. Multi-Agent Council (entirely new)

**Superpowers**: Has no equivalent. The agent picks an approach on its own.

**UltraFlow**: Dispatches 5 persona-driven agents in parallel, each proposing a different solution:

- **10 personas**: Pragmatist, Architect, Hacker, Researcher, Minimalist, Futurist, Guardian, Optimizer, User Advocate, Integrator
- **5 selected per problem** based on domain (e.g., security work gets Guardian + Pentester-minded personas)
- **Synthesis**: Top 5 options presented to user, combining pure and hybrid approaches
- **Minimum floor**: At least 3 real agents must be dispatched (not synthesized in-head)

**Why it performs better**: One perspective finds one solution. Five perspectives find the best solution. The persona diversity ensures the agent doesn't just pick the first reasonable approach — it considers simplicity, security, performance, UX, and conventions simultaneously.

### 3. Adversarial Testing (entirely new)

**Superpowers**: Has no equivalent. After implementation, it relies on the existing test suite.

**UltraFlow**: Dispatches 3 specialized "breaker" agents to actively try to destroy the code:

| Breaker | What They Attack |
|---------|-----------------|
| Chaos Monkey | Random inputs, boundary values, null/undefined, unicode, malformed data |
| Pentester | Injection, auth bypass, privilege escalation, path traversal, exposed secrets |
| Load Tester | Concurrent access, race conditions, resource exhaustion, deadlocks |
| Compatibility Checker | Cross-platform issues, version mismatches, missing dependencies |
| Regression Hunter | Runs existing tests, checks unchanged functionality still works |

**Fix-and-rebreak cycles**: When a breaker finds a critical issue, you fix it, then re-dispatch the same breaker. Only pass when all breakers report SOLID.

**Why it performs better**: Normal testing proves the happy path works. Adversarial testing proves the system is robust. This catches bugs that would otherwise reach production.

### 4. Receiving Code Review (new skill)

**Superpowers**: Has this skill. UltraFlow was initially missing it — added during audit.

**UltraFlow version**: Bans performative agreement ("You're absolutely right!"). Requires:
- Checking the codebase before agreeing with any suggestion
- YAGNI verification (grep for actual usage before adding features)
- Pushback with technical reasoning when suggestions are wrong

### 5. Anti-Hallucination Measures (new, across all skills)

**Superpowers**: Has verification-before-completion but no cross-cutting anti-hallucination strategy.

**UltraFlow additions**:

| Measure | Where | What It Prevents |
|---------|-------|-----------------|
| Subagent Verification Rule | building | Agent fabricating subagent results |
| Evidence Quoting | verification | Agent claiming tests pass without running them |
| Minimum Floors | discovery, council | Agent skipping stages by calling them "flexible" |
| Anti-Drift Protocol | using-ultraflow | Agent forgetting skills after long sessions |
| Context Injection Example | implementer | Subagents reading wrong files or hallucinating file contents |
| Banned Phrases | receiving-review | Agent agreeing performatively without technical verification |

### 6. Token Budget Awareness (new metadata)

**Superpowers**: No token awareness. Each skill loads without considering budget.

**UltraFlow**: Each skill self-reports its estimated token cost in the bootstrap table. The agent can make cost-aware decisions:
- Trivial tasks skip expensive skills (discovery, council)
- BUT rigid skills (building, debugging, verification) can NEVER be skipped for token savings

### 7. Intent Echo (v2 addition)

**Superpowers**: No equivalent. Agent interprets and executes immediately.

**UltraFlow**: Before executing ANY user request, the agent echoes back its understanding in 1 sentence: "I understand you want me to: [summary]. Correct?" User confirms or corrects. This catches misinterpretation before any work begins — the cheapest possible error prevention (costs ~20 tokens, saves thousands when it catches a misunderstanding).

### 8. Assumption Detector (v2 addition)

**Superpowers**: No equivalent. Agent builds on unchecked assumptions.

**UltraFlow**: Before starting ANY building task, the agent must list 2-3 assumptions (e.g., "file X exists", "function Y has parameter Z") and verify each against the codebase. Building on wrong assumptions compounds errors — this catches them at the cheapest point to fix.

### 9. Confidence Scoring (v2 addition)

**Superpowers**: No equivalent. All subagent outputs treated equally regardless of certainty.

**UltraFlow**: Every subagent response must include `Confidence: [1-5]`. Score 1-2 triggers re-dispatch with more context. Score 3 requires careful manual review. Score 4-5 is trusted but still independently verified. This prevents low-quality subagent outputs from silently contaminating the build.

### 10. Checkpoint Rollback (v2 addition)

**Superpowers**: No equivalent. Failed fixes compound on top of each other.

**UltraFlow**: If a task fails 2+ times, the agent runs `git stash` or `git checkout .` to return to last known good state before retrying. This prevents the common failure mode where fix attempt #2 breaks something fix attempt #1 didn't, creating a tangled mess.

### 11. Scope Creep Guard (v2 addition)

**Superpowers**: No equivalent. Agent may silently add unrequested features.

**UltraFlow**: Before claiming "done," the verification skill now asks: "Did I do ONLY what was asked? Did I add anything extra?" Unrequested features, refactoring, or comments are reverted. Scope creep is treated as a hallucination of requirements — the agent imagined a requirement that doesn't exist.

### 12. Context Decay Detection (v2 addition)

**Superpowers**: Has a basic "re-read after compaction" note.

**UltraFlow**: Tracks tasks since last skill re-read with specific thresholds:
- Every 5 tasks: re-read bootstrap silently
- Every 10 tasks: re-read ALL active skill files
- After compaction: immediate re-invoke
- If noticing step-skipping: immediate re-read ("you are drifting")

### 13. Output Fingerprinting (v2 addition)

**Superpowers**: No equivalent. Plans and specs can become stale without detection.

**UltraFlow**: Every plan ends with a 1-line "fingerprint" summarizing the expected end state. When a plan is referenced later in the session, the fingerprint is checked against current codebase state. If they don't match, the plan is stale and must be updated before proceeding.

---

## What Was Removed from Superpowers and Why

### 1. Separate TDD Skill → Merged into Building

**Why removed**: TDD was a separate skill in Superpowers (test-driven-development/SKILL.md, ~200 lines). Loading it required a separate skill invocation. In UltraFlow, TDD is the Iron Law of the building skill — it's embedded as Step 2, not a separate load. Saves ~500 tokens per workflow.

**What was preserved**: The RED-GREEN-REFACTOR cycle, the iron law ("no production code without a failing test first"), the anti-rationalization defenses.

### 2. Separate Code Review Request Skill → Merged into Building

**Why removed**: Superpowers had requesting-code-review as its own skill. In UltraFlow, inline code review is Step 3 of building — after every task's TDD cycle, a reviewer subagent is dispatched automatically. Saves ~400 tokens per workflow.

**What was preserved**: The reviewer prompt, the mandatory review after every task.

### 3. Separate Git Worktrees + Finishing Branch → Merged into Git-Flow

**Why removed**: Superpowers had two skills (using-git-worktrees, finishing-a-development-branch). These always appear together — you create a worktree to start, and finish the branch when done. One skill handles both. Saves ~300 tokens.

**What was preserved**: Worktree setup, dependency detection, baseline testing, 4 finishing options (merge/PR/keep/discard), safety checks.

### 4. Writing-Skills Meta-Skill

**Why removed**: Superpowers has a skill for writing skills (TDD applied to skill authorship). This is a meta-concern that most users never need. It added ~400 tokens of content for a rare use case.

**Trade-off**: Users who create their own skills lose this guide. Acceptable because skill authorship is rare.

### 5. Visual Companion for Brainstorming

**Why removed**: Superpowers includes a browser-based visual companion (HTML server with frame templates) for brainstorming. This adds complexity (scripts, server, HTML templates) without proportional value — most Claude Code users work in terminals, not browsers.

**What replaced it**: Option Cards in the discovery skill achieve the same goal (helping users visualize options) without requiring a separate server.

### 6. Pressure-Test Scenarios, Root-Cause-Tracing Guide, Condition-Based-Waiting Patterns

**Why removed**: These are supporting reference files in Superpowers' debugging skill (~300 lines total). They provide detailed guides for specific debugging scenarios but are rarely needed. Token cost when loaded is significant.

**Trade-off**: Power debuggers lose depth guides. The core 4-phase debugging process is fully preserved and handles 90% of cases.

### 7. Cross-Platform Support Files (AGENTS.md, GEMINI.md, Codex/Copilot adapters)

**Why removed**: UltraFlow v1 focuses on Claude Code first. Cross-platform files add complexity without benefiting the primary audience. Can be added in v2.

### 8. Deprecated Command Files

**Superpowers** has deprecated commands/ directory (brainstorm.md, execute-plan.md, write-plan.md) pointing to new skill equivalents. UltraFlow starts fresh without legacy redirects.

---

## How Token Efficiency Was Achieved

### Quantitative Comparison

| Metric | Superpowers | UltraFlow | Reduction |
|--------|------------|-----------|-----------|
| Total skills | 13 | 11 | 15% fewer |
| Avg lines per SKILL.md | ~300 | **71** | **76% shorter** |
| Max lines per SKILL.md | ~500 | **89** | **82% shorter** |
| Total corpus lines | ~5,400 | **942** | **83% fewer** |
| Total corpus tokens | ~23,400 | **~7,800** | **67% fewer** |
| Bootstrap injection | ~650 tokens | ~430 tokens | 34% smaller |
| Full workflow load | ~14,500 tokens | **~6,100 tokens** | **58% fewer** |
| Novel capabilities | 0 | **7** (v2) | N/A |
| Anti-hallucination measures | 3 | **14** | 367% more |

### Techniques Used

1. **Tighter writing**: Every sentence earns its place. No filler, no repetition of concepts the agent already knows.

2. **On-demand loading**: Supporting files (personas.md, breaker-personas.md, plan-reviewer.md, implementer.md, reviewer.md) are only loaded when their parent skill explicitly needs them. Superpowers loads supporting files eagerly with the skill.

3. **Code intent in plans**: Plans describe WHAT code does, not the full implementation. Exception: critical sections (auth, data models, API contracts) still get full code blocks.

4. **Compressed subagent prompts**: Each subagent prompt is under 50 lines with explicit word limits on responses (100-200 words). This prevents both prompt bloat and response bloat.

5. **Merged skills**: TDD + code review + execution = one building skill. Git worktrees + branch finishing = one git-flow skill. Three fewer skill invocations per workflow.

6. **Token budget metadata**: The bootstrap skill shows token costs per skill, enabling the agent to make cost-aware routing decisions without loading and discarding skills.

---

## How Hallucination and Drift Were Addressed

### Problem 1: Phantom Subagent Execution
**Risk**: The agent claims to have dispatched subagents but fabricates their responses.
**Solution**: The building skill now contains a **Subagent Verification Rule**: "After ANY subagent dispatch, you MUST independently verify their claims. Run the tests yourself. Read the diff yourself."

### Problem 2: Phantom Test Execution
**Risk**: The agent claims "tests pass" without actually running them.
**Solution**: The verification skill now requires **quoting the last 10-20 lines of output verbatim** in the response. Not a summary — actual output. If the agent can't quote it, it didn't run it.

### Problem 3: Token Budget as Skip Justification
**Risk**: The agent uses token awareness to skip rigid skills ("too expensive").
**Solution**: The bootstrap skill now explicitly states: "Token budget NEVER justifies skipping rigid skills (building, debugging, verification, receiving-review)."

### Problem 4: "Flexible" Skills as Escape Hatches
**Risk**: The agent reduces flexible skills (discovery, council, planning) to near-zero depth.
**Solution**: Each flexible skill now has a **Minimum Floor**:
- Discovery: must always complete Stages 1-2 (Domain Scan + Option Cards)
- Council: must dispatch at least 3 real agents (not synthesize in-head)
- Planning: must always run self-review

### Problem 5: Long-Session Drift
**Risk**: After 10+ tasks, the agent forgets skill protocols and reverts to default behavior.
**Solution**: The bootstrap skill now includes an **Anti-Drift Protocol**:
- "Every 10 tasks, re-read this file to recalibrate"
- "After any context compaction event, re-invoke this skill"

### Problem 6: Performative Agreement in Code Review
**Risk**: The agent agrees with every review comment without checking if they're correct.
**Solution**: The receiving-review skill **bans specific phrases** ("You're absolutely right!", "Great catch!") and requires codebase verification before agreeing with any suggestion.

### Problem 7: Implementation Divergence from Intent-Based Plans
**Risk**: When plans describe "code intent" instead of code blocks, multiple subagents interpret them differently, producing incompatible implementations.
**Solution**: The planning skill now requires **code blocks for critical sections** (auth logic, data models, API contracts, security checks, migration schemas) while allowing intent-only descriptions for straightforward code.

### Problem 8: Misinterpreted User Intent (v2)
**Risk**: Agent misunderstands user request and builds the wrong thing entirely.
**Solution**: **Intent Echo** — before executing, agent states "I understand you want me to: [X]. Correct?" Catches misinterpretation at ~20 tokens instead of discovering it after 1000+ tokens of wrong work.

### Problem 9: Building on Wrong Assumptions (v2)
**Risk**: Agent assumes a file/function/library exists and writes code that references it incorrectly.
**Solution**: **Assumption Detector** — before each building task, list and verify 2-3 assumptions against the actual codebase. Wrong assumptions are caught before code is written.

### Problem 10: Low-Confidence Subagent Outputs Accepted Silently (v2)
**Risk**: Subagent is unsure but reports DONE anyway. Orchestrator accepts without scrutiny.
**Solution**: **Confidence Scoring** — every subagent must report confidence 1-5. Low scores trigger re-dispatch or manual review rather than silent acceptance.

### Problem 11: Compounding Errors from Failed Fixes (v2)
**Risk**: Fix attempt #1 fails. Fix attempt #2 builds on #1's broken state. By attempt #3, the code is a tangled mess of partial fixes.
**Solution**: **Checkpoint Rollback** — after 2 failed attempts, `git stash`/`git checkout .` to return to last good state. Start fresh, don't compound.

### Problem 12: Agent Adds Unrequested Features (v2)
**Risk**: Agent "helpfully" adds error handling, refactoring, comments, or features that weren't asked for. This is a hallucination of requirements.
**Solution**: **Scope Creep Guard** — before claiming done, explicitly check "Did I do ONLY what was asked?" Extras get reverted.

### Problem 13: Late-Session Quality Degradation (v2)
**Risk**: After 15+ tasks, agent gradually stops following skill protocols as context fills up.
**Solution**: **Context Decay Detection** — specific thresholds (5 tasks = re-read bootstrap, 10 tasks = re-read all skills, noticing step-skipping = immediate re-read).

### Problem 14: Stale Plan References (v2)
**Risk**: Agent references a plan written 20 tasks ago. The codebase has changed since then but the plan hasn't been updated.
**Solution**: **Output Fingerprinting** — every plan has a 1-line fingerprint of expected end state. When referenced, compare against current state. Mismatch = plan is stale.

---

## Architecture Summary

```
11 skills, 25 files, ~942 total lines (v2: added 7 capabilities, reduced 46 lines)

Workflow:
  discovery (5 stages) → council (5 personas) → planning (self-reviewed)
       → building (TDD + inline review) → adversarial-testing (3 breakers)
            → verification (evidence-quoted) → git-flow (merge/PR/keep/discard)

Cross-cutting:
  debugging (4 phases) — invoked when things break
  receiving-review — invoked when feedback arrives
  parallel-dispatch — invoked for independent tasks
  using-ultraflow — bootstrap, re-read every 10 tasks
```

## File Inventory

```
ultraflow/ (v2 — 25 files, 942 skill lines)
├── package.json, LICENSE, README.md, DETAILED-EXPLANATION.md
├── .claude-plugin/plugin.json, marketplace.json
├── hooks/hooks.json, session-start
├── docs/token-budget.md
└── skills/
    ├── using-ultraflow/SKILL.md          (66 lines) — +Intent Echo, +Context Decay Detection
    ├── discovery/SKILL.md                (66 lines) — compressed templates (-34 lines)
    ├── council/SKILL.md + personas.md    (89 + 31 lines) — +Minimum Floor
    ├── planning/SKILL.md + plan-reviewer.md (72 + 31 lines) — +Fingerprinting, +Assumptions check
    ├── building/SKILL.md + implementer.md + reviewer.md (85 + 45 + 30 lines) — +Assumption Detector, +Checkpoint Rollback, +Confidence Scoring
    ├── debugging/SKILL.md                (67 lines) — +Assumption listing in Phase 1
    ├── adversarial-testing/SKILL.md + breaker-personas.md (85 + 26 lines)
    ├── verification/SKILL.md             (61 lines) — +Scope Creep Guard
    ├── receiving-review/SKILL.md         (63 lines)
    ├── git-flow/SKILL.md                 (71 lines)
    └── parallel-dispatch/SKILL.md        (54 lines) — +Confidence Scoring requirement
```
