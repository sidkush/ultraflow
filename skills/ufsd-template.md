# UFSD - UltraFlow State Document

Canonical location: `docs/ultraflow/specs/`.

Canonical filename: `UFSD-YYYY-MM-DD-<feature>.md` or `DRAFT-UFSD-YYYY-MM-DD-<feature>.md`.

Do not reference legacy non-specs UFSD locations or filename conventions.

## Summary Layer

```markdown
# UFSD Summary
Feature:
Ticket:
Entry point:
Happy path:
Scope baseline: In: [...] Out: [...]
Risk class: Feature/UI | Trust-layer
Commit tier: T1 | T2 | T3
Two-gate status: Reachability=<pass|fail|n/a>; Frontend wiring=<pass|fail|n/a>
Pipeline stage: brainstorming | council | writing-plans | execution | adversarial | verification | done
Deferred bugs link: docs/superpowers/triage/deferred-bugs.md
```

## Detail Blocks

Append, never overwrite:

```markdown
## <skill-name> - <date>
Decisions:
Assumptions:
Evidence:
Failure paths:
Operator-comprehension notes:
Open questions:
```

## Size Discipline

Keep summary under 10 lines when possible. Move long audits to `docs/ultraflow/audits/` and link them.
