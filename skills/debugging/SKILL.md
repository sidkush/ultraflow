---
name: debugging
description: Use when encountering any bug, test failure, or unexpected behavior. Systematic 4-phase approach with evidence gates, blind skeptic verification, and loop detection.
---

# Systematic Debugging

## Iron Law
**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

## Session Start — UFSD Read Protocol
Before anything else:
1. Check for UFSD summary layer at `docs/ultraflow/specs/[context].ufsd.md`. If found, extract: prior assumptions, invariants, cascade-map — use to scope Phase 1.
2. If no UFSD exists, note: "No UFSD found — starting fresh."
3. UFSD-sourced invariants go directly into Phase 1 Assumptions list (pre-verified unless contradicted by current evidence).

## Meta-Cognitive Failure Mode Map
**Generate at session start — BEFORE Phase 1.** Produce exactly 5 failure modes specific to THIS debugging session context:

| # | failure mode | Session-Specific Risk |
|---|---|---|
| 1 | Treating symptom as root cause | [why this session is at risk] |
| 2 | Confirmation bias on first hypothesis | [which prior assumption could bias] |
| 3 | Missing cascade paths | [which system areas could propagate] |
| 4 | Unverified ASSUMPTION carried forward | [which assumptions are untested here] |
| 5 | Knowledge boundary gap | [where uncertainty is highest] |

**Inject top 2 failure modes into Skeptic dispatch prompt verbatim** under heading "Watch for these failure modes."

## Entry — Choose Path First

| Signal | Path |
|--------|------|
| Obvious typo/import, 1 file, cause immediately visible | **Simple** → Phase 1 + Phase 4 only |
| Cascading errors, 2+ files, ambiguous cause, CI/env mismatch, 2+ prior fix sessions | **Complex** → All phases below |

**Auto-reclassify to Complex if:** fix requires >1 file / fails unexpectedly / error recurs after fix.
**On reclassify (Simple→Complex):** carry Phase 1 artifact verbatim into the new Complex-path Phase 1. Do not restart — existing evidence is still valid.
**Resume (prior attempts exist):** audit prior fix attempts into Phase 1 Evidence Artifact before starting — treat each as a "DENIED" hypothesis with evidence. Do not re-cover already-disproven ground.

## Phase 1: Observe
1. Reproduce: run exact failing command, copy full output
2. Read: what does the error actually say — not what you assume
3. Assumptions: list each, verify against codebase, and log to UFSD detail layer:
   `ASSUMPTION: [text] | VALIDATED: [yes/no] | SOURCE: [file:line or "inferred"]`
   Unvalidated assumptions at end of session = risk items — list explicitly in UFSD Write.
4. Isolate: smallest input that triggers the failure
5. Timeline: `git log --oneline -10`

**Evidence Artifact** — required before Phase 2. No artifact = cannot advance.
```
Command: [exact command]
Output: [verbatim, 5-20 lines]
  OR (silent failure): Behavioral diff: [expected] vs [actual] — observable difference: [measurable]
Assumptions verified: [each + how checked]
UFSD sourced: [yes — invariants used: X | no]
```

## Phase 2: Hypothesize
Form **exactly 2 mechanism-level** hypotheses, each citing quoted Phase 1 evidence:
- H1: `[X] fails BECAUSE [invariant/precondition] is violated BY [caller/condition]` — evidence: "[quoted line]"
- H2: `[X] fails BECAUSE [invariant/precondition] is violated BY [caller/condition]` — evidence: "[quoted line]"

**Symptom-level rejected**: "X returns null" ✗ → "X returns null BECAUSE caller Z skips init guard, violating X's precondition that cache is populated" ✓
Intuition without quoted evidence = restart Phase 1.

**Label each hypothesis:** `ROOT_CAUSE` or `SYMPTOM`. Symptom-level hypotheses are auto-rejected — do not advance to Phase 3.

**Counterfactual Gate — required before accepting any hypothesis:**
For each hypothesis, state its strongest counterargument with evidence before acceptance:
```
H1 Counterfactual: "The strongest evidence AGAINST H1 is [X — cite specific line/measurement/behavior]."
H1 Accepted because: "[Y defeats X — must cite code, test, or measurement (MVP-Proof)]."
H2 Counterfactual: "The strongest evidence AGAINST H2 is [X — cite specific line/measurement/behavior]."
H2 Accepted because: "[Y defeats X — must cite code, test, or measurement (MVP-Proof)]."
```
If Y cannot pass MVP-Proof (no code, test, or measurement citation) → hypothesis unacceptable → return to Phase 1 with new evidence.

## Ghost Check Protocol
Run **before Phase 3** (label `Ghost α`) AND **before Phase 4** (label `Ghost β`). Cannot advance without this block.
```
Ghost [α/β]1: [alternative explanation] — ruled out because [specific reason]
Ghost [α/β]2: [alternative explanation] — ruled out because [specific reason]
```
Minimum 2 alternatives. Exception — only if 2nd is **structurally impossible** (meaning: architecture, language constraints, or error type makes it mechanically impossible — not merely improbable):
`Ghost [α/β] — one alternative: [explanation] — ruled out: [reason] — 2nd impossible: [structural reason]`
Zero alternatives is always a violation.

## Phase 3: Test
Design a **causally tied** test per hypothesis — the test must directly probe the proposed mechanism, not produce a coincidental pass.

**Write the Prediction BEFORE running the test.** CONFIRMED without a prior Prediction block = retrospective confirmation = Red Flag violation.
```
Prediction: IF [mechanism from H1/H2] is correct, THEN [exact command] will produce [exact output/exit code/state change]. Anything else → DENIED.
Test command: [exact]
Output: [verbatim, 5-20 lines]
H1: CONFIRMED/DENIED — [reason — CONFIRMED requires prediction to match observed output]
H2: CONFIRMED/DENIED — [reason — CONFIRMED requires prediction to match observed output]
```
Both denied → return to Phase 1 with new evidence. No 3rd hypothesis without fresh evidence.
**Multi-causal:** If H1 CONFIRMED and fixed but symptoms persist → remaining symptoms = new bug. Carry Phase 1 artifact forward as context — do not discard history.

## Pattern Exhaustion — after CONFIRMED, before Fix
Extract the broken mechanism as an abstract pattern. Scan codebase for every location sharing it:
```
Pattern: [abstract broken assumption — e.g., "caller assumes cache is populated without checking"]
Affected locations:
  - [file:line] — [why it shares this pattern]
  (or: "None found — verified by: [grep/search command + verbatim output]")
```
**Fix session must cover ALL listed locations.** Skipping any = Skeptic will REPRODUCED.

## Phase 4: Fix
Only after root cause confirmed, Pattern Exhaustion complete, and Ghost β complete:
1. Write failing test (RED)
2. **Open Fix Session** — make all related edits atomically
3. **Close Fix Session** — run full test suite

**Evidence Artifact** — required before Skeptic dispatch:
```
Session: [files changed + 1-line summary each]
Test: [command + verbatim result]
Regression: [suite command + result]
Coverage matrix (bug repro variants — NOT fix details):
  - Variant 1 (boundary): [command/input] — triggers bug if not fixed
  - Variant 2 (null/invalid): [command/input] — triggers bug if not fixed
  - Variant 3 (adjacent path): [command/input] — triggers bug if not fixed
Receipts (both required — missing either = cannot dispatch Skeptic):
  Pre-fix:  [command] → exit [N] → [failure line verbatim]   ← MUST show failure
  Post-fix: [command] → exit [N] → [pass line verbatim]      ← MUST show pass
Counterexample: [input/state that would reproduce original bug and is now caught by the new test]
  Cannot produce one → write "POSITIONAL FIX — escalate before closing session"
```

**Pre-Skeptic Confidence** (required — fill before dispatching):
```
Fix depth [1-5]:  1=symptom patched only  5=root invariant enforced at all boundaries
Paths covered [1-5]:  1=original repro only  5=all Pattern Exhaustion locations confirmed fixed
Session penalty: −1 per prior failed session on this bug (sessions so far: N)
Score = depth + paths − penalty  [min 1]
```
Score ≤4 → write: "Causal justification: [why this fix is complete despite low score]" before dispatching.

4. Dispatch `skeptic` subagent **once per session** (see `skeptic.md`).
   Pass: Phase 1 Evidence Artifact **verbatim** + Coverage matrix + environment context + top 2 failure modes from Meta-Cognitive Failure Mode Map. Never pass fix details.
   **Intermittent bugs:** note in dispatch — Skeptic runs repro steps 3× and reports all outcomes.

**Status before Skeptic sign-off — only these phrases permitted:**
- `"Awaiting Skeptic sign-off"`
- `"Fix session closed — Skeptic dispatched"`
- `"Skeptic returned [verdict] — [next action]"`

If Skeptic → `BUG NOT DETERMINABLE`: improve Phase 1 artifact and re-dispatch once.
If `BUG NOT DETERMINABLE` twice consecutive → STOP, escalate to user.

## UFSD Write Protocol
At end of Phase 4, after Skeptic sign-off, write to `docs/ultraflow/specs/[context].ufsd.md`:

**Summary layer** (append one line):
`approach=[fix-type] | confidence=[pre-skeptic score] | session=[date] | outcome=[RESOLVED/ESCALATED]`

**Detail layer** (append block):
```
## Debug Session [date]
Decisions: [key Phase 2 hypothesis choices and why]
Fix summary: [what changed and why it addresses root cause]
Assumption outcomes:
  - ASSUMPTION: [text] | VALIDATED: [yes/no] | IMPACT: [affected fix direction / no impact]
Unvalidated assumptions (risk items): [list or "none"]
Cascade paths verified: [locations from Pattern Exhaustion + verification method]
```

## Loop Detector
Fires on **any one** of:
- Same hypothesis wording in 2nd fix session (paraphrasing counts)
- Same semantic diff in 2nd fix session — "same" means: logical operation is identical regardless of variable names, whitespace, or formatting (not just same file/line)
- **Hard cap:** 3+ fix sessions without Skeptic sign-off — fires regardless

**Circuit breaker:** if already inside a council-spawned debugging session → Loop Detector fires immediately, no recursive council dispatch.

**When fired:** STOP → dispatch `ultraflow:council` with Stuck Report (sessions: N | tried: X | failed: Y | best hypothesis: Z) → if council fails → ask user.

## Red Flags + Anti-Patterns
- Proposing a fix before Phase 2 root cause
- Retrospective confirmation — writing CONFIRMED without a prior Prediction block (the prediction must be written before the test runs)
- Accepting a hypothesis without completing the Counterfactual Gate
- Labeling a SYMPTOM-level hypothesis as ROOT_CAUSE
- "Let me try this..." without evidence (shotgun debugging)
- Unrelated changes in one Fix Session (related atomic edits are correct — shotgun multi-topic is not)
- Fixing symptoms, not causes
- Any status language not in the allowlist above
- try/catch to hide errors | commenting out failing tests | "works on my machine" without investigation
- Skipping UFSD Read at session start when a UFSD file exists
- Leaving unvalidated ASSUMPTIONs without logging them as risk items in UFSD Write
