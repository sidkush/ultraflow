# UltraFlow

Enhanced agentic skills framework for coding agents. Better brainstorming, smarter decisions, adversarial testing, TDD-enforced debugging with post-mortem journals — in ~55% fewer tokens than alternatives.

## What's Different

| Feature | Traditional | UltraFlow |
|---------|------------|-----------|
| Requirements gathering | Linear Q&A | **Progressive Discovery** — 5-stage funnel with option cards, counterfactuals, and assumption registry |
| Decision making | Single perspective | **20-Persona Council** — all 20 decision intelligence personas dispatched every run, synthesized into theme clusters with evidence triangulation |
| Implementation | Ad-hoc coding | **TDD-Enforced Building** — RED→GREEN→REFACTOR with inline code review, invariant preservation, and skill chain interrupts |
| Debugging | "Try this fix" loops | **Session-Scoped TDD Fix Loop** — max 3 bugs/session, one at a time, proof-of-fix gates, blind skeptic verification, mandatory debug journals |
| Security testing | Unit + integration | **20 Security Analysts** — 7 specialized clusters (injection, boundaries, concurrency, resources, integration, logic, crypto) with fix-and-rebreak cycles |
| Token usage | ~12,000-18,000 per workflow | **~5,500-8,000** per workflow (~55% savings) |

## Skills

### Core Workflow (in order)
1. **discovery** — Progressive requirement discovery in 5 stages: domain scan, option cards (with counterfactuals), deep dive, spec preview with assumption registry, confidence check with coverage scoring. Writes UFSD spec file.
2. **council** — All 20 decision intelligence personas (Contrarian, Actuarian, Archaeologist, Synthesizer, Economist, Anthropologist, Regulator, Migrationist, Epidemiologist, Cognitive Load Auditor, Chronologist, Measurement Skeptic, Analogist, Scope Prosecutor, Debt Collector, Operator, Build/Buy Arbitrageur, Privacy Engineer, Falsificationist, Velocity Accountant) dispatched in parallel every run. Synthesized into 3-5 theme clusters with evidence triangulation, counterfactual gate, and assumption registry. Also has 10 specialist investigators (personas 21-30) for debugging/root cause analysis.
3. **planning** — Implementation plans with 2-5 minute task granularity, exact file paths, test commands, invariant preservation, assumption registry, counterfactual gate, and MVP-proof. Self-reviewed by plan-reviewer subagent against failure mode map.
4. **building** — TDD-enforced (RED→GREEN→REFACTOR per task) with invariant preservation checks after each GREEN. Inline code review via reviewer subagent. Implementer subagent for complex tasks. Skill chain interrupts (LOCAL/UPSTREAM/HALT) on failure.
5. **adversarial-testing** — All 20 security analysts dispatched in parallel across 7 clusters: Input Validation (injection, auth, CSRF), Boundary Testing (numeric, encoding, structure), Concurrency (race conditions, crash recovery, session state), Resource (memory, CPU complexity, throughput), Integration (API contracts, network partition, config drift), Logic (business logic, state machines, temporal/precision), Deep Specialists (crypto, regression). Anti-hallucination gate, evidence triangulation, severity×blast-radius priority matrix, fix-and-rebreak cycles.
6. **verification** — Evidence-gated completion claims. Must run commands fresh, quote verbatim output (command + exit code + result + full output), scope creep guard. Bans "should", "probably", "I think".
7. **git-flow** — Worktree creation (auto-detect deps, baseline test, gitignore check) + branch finishing (merge, push+PR, keep, or discard with confirmation).

### Supporting Skills
8. **debugging** — 5-phase systematic approach: observe (reproduce + assumptions), hypothesize (mechanism-level + counterfactual gate), test (prediction-first), TDD bug fix loop (RED confirmation → minimal fix → GREEN confirmation → regression check → proof-of-fix gate → blind skeptic), debug journal (mandatory post-mortem in `docs/ultraflow/debug-journals/`). Session-scoped: max 3 bugs, one at a time, sequential completion only. Loop detector escalates to council after 3 failed sessions.
9. **parallel-dispatch** — Dispatches 2-5 focused agents simultaneously for independent tasks. Conflict detection, confidence-based re-dispatch, and integrated test verification.
10. **receiving-review** — Technical verification of code review feedback before implementing. Bans performative agreement. Each suggestion gets AGREE+IMPLEMENT, AGREE+DEFER, PUSHBACK (with counter-evidence), or CLARIFY. YAGNI check on additions.
11. **using-ultraflow** — Bootstrap and skill router. Establishes instruction priority, mandatory intent echo, anti-drift protocol (re-read skills every 5 tasks), and token budget awareness. Enforces rigid/flexible skill types.

## Workflow

```
User request
    |
    v
[discovery] — 5 stages: domain scan → option cards → deep dive → spec → confidence
    |          Writes UFSD spec with scope baseline + assumption registry
    v
[council] — 20 personas propose solutions in parallel
    |         Synthesized into 3-5 theme clusters with evidence triangulation
    v
[planning] — Precise task breakdown with invariant preservation
    |          Self-reviewed by plan-reviewer subagent
    v
[building] — TDD cycle per task (RED → GREEN → REFACTOR)
    |          Inline code review + invariant checks after each task
    v
[adversarial-testing] — 20 security analysts in 7 clusters attack
    |                     Fix-and-rebreak cycles, priority triage
    v
[verification] — Fresh evidence of success (verbatim output required)
    |
    v
[git-flow] — Merge, PR, or keep

Supporting:
  [debugging]          — invoked on any bug/failure (session-scoped TDD fix loop)
  [parallel-dispatch]  — invoked for independent tasks within building
  [receiving-review]   — invoked when receiving PR/code review feedback
  [using-ultraflow]    — bootstrap at conversation start
```

## Token Efficiency

UltraFlow is designed for **minimum token consumption**:

- **11 skills** covering the full development lifecycle
- **Lazy-loaded** supporting files (personas, breaker profiles, subagent prompts loaded on-demand only)
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
│   │   └── personas.md          (20 decision + 10 specialist personas)
│   ├── planning/
│   │   ├── SKILL.md
│   │   └── plan-reviewer.md
│   ├── building/
│   │   ├── SKILL.md
│   │   ├── implementer.md
│   │   └── reviewer.md
│   ├── debugging/
│   │   ├── SKILL.md
│   │   ├── skeptic.md
│   │   └── debug-journal-template.md
│   ├── adversarial-testing/
│   │   ├── SKILL.md
│   │   └── breaker-personas.md  (20 analysts in 7 clusters)
│   ├── verification/SKILL.md
│   ├── git-flow/SKILL.md
│   ├── parallel-dispatch/SKILL.md
│   ├── receiving-review/SKILL.md
│   └── ufsd-template.md
└── docs/
    └── token-budget.md
```

## Cross-Skill State: UFSD

All skills read and write to a shared **UltraFlow State Document** (UFSD):
- **Specs**: `docs/ultraflow/specs/UFSD-YYYY-MM-DD-<feature>.md`
- **Plans**: `docs/ultraflow/plans/YYYY-MM-DD-<feature>.md`
- **Debug Journals**: `docs/ultraflow/debug-journals/YYYY-MM-DD-<slug>.md`

The UFSD carries scope baselines, assumptions, invariants, decisions, and risk items across the full workflow so downstream skills inherit context without re-reading upstream artifacts.

## Installation

### Claude Code (Recommended)

```bash
# Install as a plugin from GitHub
claude install-plugin github:sidkush/ultraflow
```

### Manual Installation

```bash
# Clone the repo
git clone https://github.com/sidkush/ultraflow.git

# Navigate to it
cd ultraflow

# Install as Claude Code plugin (from the repo directory)
claude plugin add .
```

### Alternative: Copy to Skills Directory

```bash
# Clone
git clone https://github.com/sidkush/ultraflow.git

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
/council      # Launch 20-persona decision council
/building     # Begin TDD implementation
/debugging    # Systematic bug fix with TDD loop
```

## Comparison with Superpowers

| Aspect | Superpowers | UltraFlow |
|--------|------------|-----------|
| Brainstorming | Linear Q&A, one question at a time | 5-stage progressive discovery with option cards + counterfactuals |
| Decision making | Single approach | Council of 20 personas → 3-5 theme clusters with evidence triangulation |
| Debugging | "Try this fix" loops | Session-scoped TDD fix loop + proof gates + blind skeptic + debug journals |
| Security testing | Manual test suite runs | 20 security analysts in 7 clusters with fix-rebreak cycles |
| TDD | Separate skill (extra load) | Embedded in building skill + debugging skill |
| Code review | Separate skill (extra load) | Inline in building skill + dedicated receiving-review skill |
| Git worktrees + finishing | 2 separate skills | 1 merged git-flow skill |
| Token cost | ~12K-18K per workflow | ~5.5K-8K per workflow |

## Contributing

1. Fork the repo
2. Create a feature branch
3. Follow UltraFlow's own workflow (use the skills!)
4. Submit a PR

## License

MIT
