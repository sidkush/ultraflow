# Token Budget Reference

## Per-Skill Load Cost

| Skill | SKILL.md Lines | Est. Tokens | Supporting Files |
|-------|---------------|-------------|-----------------|
| using-ultraflow | ~65 | ~300 | none |
| discovery | ~110 | ~800 | none |
| council | ~95 | ~300 | personas.md (~400 on-demand) |
| planning | ~85 | ~600 | plan-reviewer.md (~200 on-demand) |
| building | ~105 | ~700 | implementer.md (~150), reviewer.md (~100) on-demand |
| debugging | ~95 | ~500 | none |
| adversarial-testing | ~100 | ~400 | breaker-personas.md (~300 on-demand) |
| verification | ~70 | ~300 | none |
| git-flow | ~95 | ~400 | none |
| parallel-dispatch | ~70 | ~300 | none |

## Full Workflow Estimate

| Phase | Tokens |
|-------|--------|
| Bootstrap (using-ultraflow) | 300 |
| Discovery (5 stages) | 800 + ~200/question |
| Council (5 personas) | 300 + 1500 + 500 synthesis |
| Planning + review | 600 + 200 + 200 |
| Building (per task) | 700 + 300 implementer + 100 reviewer |
| Adversarial (3 breakers) | 400 + 900 |
| Verification | 300 |
| Git-flow | 400 |
| **Full workflow total** | **~5,500 - 8,000** |

## Comparison with Superpowers

| Metric | Superpowers | UltraFlow | Savings |
|--------|------------|-----------|---------|
| Number of skills | 13 | 10 | 23% |
| Avg lines per SKILL.md | ~300 | ~90 | 70% |
| Full workflow tokens | ~12,000-18,000 | ~5,500-8,000 | ~55% |
| Supporting files loaded | always with skill | on-demand only | variable |
