# UltraFlow

Enhanced agentic skills framework for coding agents. Better brainstorming, smarter decisions, adversarial testing — in ~55% fewer tokens than alternatives.

## What's Different

| Feature | Traditional | UltraFlow |
|---------|------------|-----------|
| Requirements gathering | Linear Q&A | **Progressive Discovery** — staged funnel with option cards. Users find their own requirements. |
| Decision making | Single perspective | **Multi-Agent Council** — 5 persona-driven agents propose diverse solutions, synthesize top 5 options |
| Testing | Unit + integration | **Adversarial Testing** — breaker agents actively try to hack/break your code |
| Token usage | ~12,000-18,000 per workflow | **~5,500-8,000** per workflow (~55% savings) |
| Skill count | 13 separate skills | **10 merged skills** (less overhead, fewer context switches) |

## Skills

1. **discovery** — Progressive requirement discovery in 5 stages: domain scan, option cards, deep dive, spec preview, confidence check
2. **council** — 10 personas (Pragmatist, Architect, Hacker, Researcher, Minimalist, Futurist, Guardian, Optimizer, User Advocate, Integrator). Selects 5 per problem, synthesizes top 5 approaches
3. **planning** — Streamlined plans with code intent (not full blocks), self-review via subagent
4. **building** — Unified TDD + execution + inline code review. Test-first discipline enforced
5. **debugging** — 4-phase systematic approach: observe, hypothesize, test, fix. Root cause first
6. **adversarial-testing** — 5 breaker personas (Chaos Monkey, Pentester, Load Tester, Compatibility Checker, Regression Hunter). Fix-and-rebreak cycles
7. **verification** — Evidence before assertions. Fresh command execution required before any success claim
8. **git-flow** — Worktree creation + branch finishing in one skill
9. **parallel-dispatch** — Focused parallel agent dispatch with conflict detection
10. **using-ultraflow** — Bootstrap and skill router

## Installation

### Claude Code (Recommended)

```bash
# Install as a plugin from GitHub
claude install-plugin github:ultraflow/ultraflow
```

### Manual Installation

```bash
# Clone the repo
git clone https://github.com/ultraflow/ultraflow.git

# Navigate to it
cd ultraflow

# Install as Claude Code plugin (from the repo directory)
claude plugin add .
```

### Alternative: Copy to Skills Directory

```bash
# Clone
git clone https://github.com/ultraflow/ultraflow.git

# Copy skills to your Claude Code skills directory
cp -r ultraflow/skills/* ~/.claude/skills/

# Copy hooks
cp ultraflow/hooks/hooks.json ~/.claude/hooks.json
cp ultraflow/hooks/session-start ~/.claude/hooks/session-start
chmod +x ~/.claude/hooks/session-start
```

### Verify Installation

Start a new Claude Code session. You should see UltraFlow's bootstrap message. Try:

```
/discovery    # Start progressive requirement discovery
/council      # Launch multi-agent council for decisions
/building     # Begin TDD implementation
```

## Workflow

```
User request
    |
    v
[discovery] — 5 stages of progressive refinement
    |          User goes from vague → crystal clear
    v
[council] — 5 persona agents propose solutions
    |         Synthesize into top 5 options
    v
[planning] — Precise task breakdown
    |          Self-reviewed by subagent
    v
[building] — TDD cycle per task
    |          Inline code review after each
    v
[adversarial-testing] — 3 breaker agents attack
    |                     Fix-rebreak cycles
    v
[verification] — Fresh evidence of success
    |
    v
[git-flow] — Merge, PR, or keep
```

## Token Efficiency

UltraFlow is designed for **minimum token consumption**:

- **10 skills** instead of 13 (merged overlapping concerns)
- **<200 lines** per SKILL.md (vs <500 in alternatives)
- **Lazy-loaded** supporting files (personas, breaker definitions only loaded when needed)
- **Code intent** in plans, not full code blocks (~50% savings)
- **Compressed subagent prompts** (<50 lines each, explicit word limits on responses)
- **Token budget metadata** in each skill so the agent can make cost-aware decisions

Full workflow: **~5,500-8,000 tokens** for skill content.

See `docs/token-budget.md` for detailed breakdown.

## Project Structure

```
ultraflow/
├── package.json
├── README.md
├── LICENSE (MIT)
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── hooks/
│   ├── hooks.json
│   └── session-start
├── skills/
│   ├── using-ultraflow/SKILL.md
│   ├── discovery/SKILL.md
│   ├── council/
│   │   ├── SKILL.md
│   │   └── personas.md
│   ├── planning/
│   │   ├── SKILL.md
│   │   └── plan-reviewer.md
│   ├── building/
│   │   ├── SKILL.md
│   │   ├── implementer.md
│   │   └── reviewer.md
│   ├── debugging/SKILL.md
│   ├── adversarial-testing/
│   │   ├── SKILL.md
│   │   └── breaker-personas.md
│   ├── verification/SKILL.md
│   ├── git-flow/SKILL.md
│   └── parallel-dispatch/SKILL.md
└── docs/
    └── token-budget.md
```

## Comparison with Superpowers

| Aspect | Superpowers | UltraFlow |
|--------|------------|-----------|
| Brainstorming | Linear Q&A, one question at a time | 5-stage progressive discovery with option cards |
| Decision making | Single approach | Council of 5 personas → top 5 options |
| Regression testing | Manual test suite runs | Adversarial breaker agents with fix-rebreak cycles |
| TDD | Separate skill (extra load) | Embedded in building skill |
| Code review | Separate skill (extra load) | Inline in building skill |
| Git worktrees + finishing | 2 separate skills | 1 merged git-flow skill |
| Token cost | ~12K-18K per workflow | ~5.5K-8K per workflow |
| Skill count | 13 | 10 |

## Contributing

1. Fork the repo
2. Create a feature branch
3. Follow UltraFlow's own workflow (use the skills!)
4. Submit a PR

## License

MIT
