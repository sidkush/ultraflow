# UFSD — UltraFlow State Document

Two-layer cross-skill state propagation. Every skill reads the Summary on entry, writes Summary + Detail on exit.

## Summary Layer (≤3 lines, ≤50 tokens)

Read by all downstream skills for cheap context. Key-value format:
```
feature=[name] | approach=[decision] | scope=[1-line boundary]
risk=[top risk] | confidence=[1-5] | open=[unresolved items]
assumptions=[comma-list] | invariants=[comma-list]
```

**Rules**: Updated (not appended) after every skill. Only the latest state matters.

## Detail Layer (≤20 lines per skill)

Append-only prose sections. One block per skill that has executed:
```
## [skill-name] — [date]
### Decisions
- [decision] — Rationale: [reason]
### Assumptions Logged
- ASSUMPTION: [text] | VALIDATED: [yes|no|pending]
### Findings
- [finding — with evidence per MVP-Proof]
### Open Questions
- [question]
```

**Rules**: Appended, never overwritten. Each skill adds one block. Total UFSD ≤150 lines after full pipeline.

## Size Caps (Enforced)

| Layer | Cap | Enforcement |
|-------|-----|-------------|
| Summary | 3 lines, ≤50 tokens | Skill must truncate before writing |
| Detail per skill | ≤20 lines | Skill must compress if over |
| Total UFSD | ≤150 lines | Final skill checks and warns if exceeded |

## UFSD Lifecycle

1. **Discovery** creates the UFSD file — writes first Summary + Detail block
2. **Council** reads Summary, writes updated Summary + Detail
3. **Planning** reads Summary, writes updated Summary + Detail
4. **Building** reads Summary, writes updated Summary + Detail
5. **Adversarial** reads Summary, writes updated Summary + Detail
6. **Debugging** reads Summary (if invoked mid-pipeline), writes Detail

## File Location

UFSD lives at: `docs/ultraflow/ufsd/YYYY-MM-DD-<feature>-ufsd.md`
One UFSD per feature. Deleted or archived after feature ships.
