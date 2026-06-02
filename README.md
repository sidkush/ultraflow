# UltraFlow

AskDB-customized agentic workflow skills for high-rigor software work. UltraFlow keeps skills on, uses the `ultraflow:` namespace, and bakes in AskDB's trust-layer doctrine: value check, reachability, runtime probes, invocation-surface verification, TDD, two-stage review, adversarial pressure, and evidence-gated completion.

## What Changed In This Fork

- Operator-facing names now match the skills the operator types.
- `building` was split into `test-driven-development` and `subagent-driven-development`.
- `requesting-code-review` is a first-class skill.
- `council` is v2 with mandatory reachability and frontend-wiring gates.
- `adversarial-testing` keeps v3's six cross-cutting hunters, subsystem dispatch map, SPEC/PLAN mode, and frontend-coherence hunter.
- `audit` adds the 4+1 read-only audit workflow.
- UFSD is canonical at `docs/ultraflow/specs/`.
- Git flow no longer auto-creates worktrees or branches.
- Token claims are conservative; rigor is prioritized over the old savings target.

## Skills

### Core Flow

1. **using-ultraflow** - conversation spine, model routing, reasoning-effort header, value check, skill routing, and AskDB doctrine.
2. **brainstorming** - entry point, happy path, failure-state UX, acceptance criteria, and out-of-scope before planning.
3. **council** - council v2 decision review with Gate A Reachability/Proof-of-Life and Gate B Frontend-Wiring/Coherence.
4. **writing-plans** - implementation plans with runtime probe, invocation-surface check, operator-flow acceptance, rollback/risk, and comprehension gates.
5. **test-driven-development** - RED/GREEN/REFACTOR, AskDB preflight, trust-layer branch, and no live LLM/API tests.
6. **subagent-driven-development** - plan execution with fresh implementers and mandatory spec-compliance review before code-quality review.
7. **requesting-code-review** - reviewer dispatch template with Critical/Important/Minor triage.
8. **adversarial-testing** - v3 defensive pressure in CODE, SPEC/PLAN, or TRUST mode, sharing council's two gates.
9. **verification-before-completion** - evidence before completion claims, with UI smoke boundary.
10. **finishing-a-development-branch** - branch-in-place finish flow; merge, push/PR, keep, or discard.

### Supporting

- **systematic-debugging** - root-cause-first debugging with reachability, runtime probe, invocation-surface checks, TDD bug fix, skeptic, and journal.
- **dispatching-parallel-agents** - focused parallel agents for independent tasks and audit areas.
- **receiving-review** - verifies review feedback before implementing it.
- **audit** - 4+1 read-only audit every 4-5 commits or before risky continuation.

## Workflow

```text
using-ultraflow
  -> brainstorming
  -> council
  -> writing-plans
  -> test-driven-development / subagent-driven-development
  -> adversarial-testing
  -> verification-before-completion
  -> finishing-a-development-branch

Supporting:
  systematic-debugging
  dispatching-parallel-agents
  requesting-code-review
  receiving-review
  audit
```

UI/UX work routes through taste, impeccable, ui-ux-pro-max, or frontend-design first, then returns to UltraFlow for planning, execution, review, and verification gates.

## Derived Rename Mapping

| Previous UltraFlow skill | New operator-facing skill |
|---|---|
| discovery | brainstorming |
| planning | writing-plans |
| building | test-driven-development + subagent-driven-development |
| debugging | systematic-debugging |
| verification | verification-before-completion |
| git-flow | finishing-a-development-branch |
| parallel-dispatch | dispatching-parallel-agents |
| missing | requesting-code-review |
| missing | audit |

## Council v2 Gates

**GATE A - Reachability/Proof-of-Life:** a persona names the live entry path by grep/trace to the targeted code; dead code, non-existent APIs, or unreachable inputs cannot be CONFIRMED.

**GATE B - Frontend-Wiring/Coherence:** a persona names the frontend seam that consumes the backend change, confirms payload-shape match, and accounts for loading, error, empty, stale, and agent-editability states.

Council and adversarial testing use identical gate wording.

## Adversarial v3

Every run dispatches six cross-cutting hunters: silent-wrong-answer, silent-noop/fail-open, tenant/schema isolation, concurrency/TOCTOU, payload leak, and determinism/dead-code-green. A dispatch map adds only the subsystem clusters the surface touches, plus frontend-coherence in UI CODE mode and always in SPEC/PLAN mode.

## Cross-Skill State

All skills read/write UFSD at:

`docs/ultraflow/specs/UFSD-YYYY-MM-DD-<feature>.md`

Do not reference legacy non-specs UFSD locations or filename conventions.

Long audit outputs go to:

`docs/ultraflow/audits/`

Deferred bug tracking remains:

`docs/superpowers/triage/deferred-bugs.md`

## Project Structure

```text
ultraflow/
  .claude-plugin/
    plugin.json
    marketplace.json
  docs/
    token-budget.md
  hooks/
    hooks.json
    session-start
  skills/
    adversarial-testing/
    audit/
    brainstorming/
    council/
    dispatching-parallel-agents/
    finishing-a-development-branch/
    receiving-review/
    requesting-code-review/
    subagent-driven-development/
    systematic-debugging/
    test-driven-development/
    using-ultraflow/
    verification-before-completion/
    writing-plans/
    ufsd-template.md
```

## Installation

```bash
git clone https://github.com/sidkush/ultraflow.git
cd ultraflow
claude plugin add .
```

Manual skill copy:

```bash
cp -r ultraflow/skills/* ~/.claude/skills/
```

## Token Discipline

UltraFlow now optimizes for correctness first. Full workflows typically load more than the old savings claim when council v2, audit, adversarial v3, or trust-layer gates are active. See `docs/token-budget.md` for current estimates.

## Contributing

Use UltraFlow's own workflow. Keep skills concise, evidence-bound, and resistant to rationalization.

## License

MIT
