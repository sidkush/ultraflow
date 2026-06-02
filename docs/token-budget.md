# Token Budget Reference

UltraFlow prioritizes rigor over minimum token count. The old blanket savings claim is retired.

## Per-Skill Load Cost

| Skill | Est. tokens | Supporting files |
|---|---:|---|
| using-ultraflow | 1,400 | none |
| brainstorming | 900 | none |
| council | 1,300 | personas.md (~900) |
| writing-plans | 1,000 | plan-reviewer.md (~500) |
| test-driven-development | 900 | none |
| subagent-driven-development | 1,000 | implementer/spec/code-review prompts (~900) |
| requesting-code-review | 500 | code-reviewer.md (~500) |
| systematic-debugging | 1,300 | skeptic + journal template (~700) |
| adversarial-testing | 1,300 | breaker-personas.md (~1,800) |
| dispatching-parallel-agents | 600 | none |
| audit | 700 | none |
| verification-before-completion | 600 | none |
| finishing-a-development-branch | 500 | none |
| receiving-review | 500 | none |

## Workflow Estimates

| Flow | Estimate |
|---|---:|
| Feature/UI, planned and executed | 5,000-8,000 tokens |
| Trust-layer plan + execution gates | 8,000-14,000 tokens |
| Council v2 with personas | 3,000-5,000 tokens |
| Adversarial CODE/SPEC mode with v3 hunters | 4,500-8,000 tokens |
| Adversarial TRUST mode | 5,000-8,500 tokens |
| 4+1 audit | 3,500-6,000 tokens |

## Discipline

- Load supporting files only when the skill invokes them.
- Write long audits to `docs/ultraflow/audits/` and return path plus <=5-line summary.
- Focused tests inside folds; full floor once at the end.
- Token budget never justifies skipping TDD, reachability, review, or verification gates.
