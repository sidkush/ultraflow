---
name: debugging
description: Use when encountering any bug, test failure, or unexpected behavior. Systematic 4-phase approach with evidence gates, blind skeptic verification, and loop detection.
---

# Systematic Debugging

## Iron Law
**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

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
3. Assumptions: list each and verify against codebase
4. Isolate: smallest input that triggers the failure
5. Timeline: `git log --oneline -10`

**Evidence Artifact** — required before Phase 2. No artifact = cannot advance.
```
Command: [exact command]
Output: [verbatim, 5-20 lines]
  OR (silent failure): Behavioral diff: [expected] vs [actual] — observable difference: [measurable]
Assumptions verified: [each + how checked]
```

## Phase 2: Hypothesize
Form **exactly 2 mechanism-level** hypotheses, each citing quoted Phase 1 evidence:
- H1: `[X] fails BECAUSE [invariant/precondition] is violated BY [caller/condition]` — evidence: "[quoted line]"
- H2: `[X] fails BECAUSE [invariant/precondition] is violated BY [caller/condition]` — evidence: "[quoted line]"

**Symptom-level rejected**: "X returns null" ✗ → "X returns null BECAUSE caller Z skips init guard, violating X's precondition that cache is populated" ✓
Intuition without quoted evidence = restart Phase 1.

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
Design a **causally tied** test per hypothesis — the test must directly probe the proposed mechanism, not produce a coincidental pass. Run it:
```
Test command: [exact]
Output: [verbatim, 5-20 lines]
H1: CONFIRMED/DENIED — [reason]
H2: CONFIRMED/DENIED — [reason]
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
   Pass: Phase 1 Evidence Artifact **verbatim** + Coverage matrix + environment context. Never pass fix details.
   **Intermittent bugs:** note in dispatch — Skeptic runs repro steps 3× and reports all outcomes.

**Status before Skeptic sign-off — only these phrases permitted:**
- `"Awaiting Skeptic sign-off"`
- `"Fix session closed — Skeptic dispatched"`
- `"Skeptic returned [verdict] — [next action]"`

If Skeptic → `BUG NOT DETERMINABLE`: improve Phase 1 artifact and re-dispatch once.
If `BUG NOT DETERMINABLE` twice consecutive → STOP, escalate to user.

## Loop Detector
Fires on **any one** of:
- Same hypothesis wording in 2nd fix session (paraphrasing counts)
- Same semantic diff in 2nd fix session — "same" means: logical operation is identical regardless of variable names, whitespace, or formatting (not just same file/line)
- **Hard cap:** 3+ fix sessions without Skeptic sign-off — fires regardless

**Circuit breaker:** if already inside a council-spawned debugging session → Loop Detector fires immediately, no recursive council dispatch.

**When fired:** STOP → dispatch `ultraflow:council` with Stuck Report (sessions: N | tried: X | failed: Y | best hypothesis: Z) → if council fails → ask user.

## Red Flags + Anti-Patterns
- Proposing a fix before Phase 2 root cause
- "Let me try this..." without evidence (shotgun debugging)
- Unrelated changes in one Fix Session (related atomic edits are correct — shotgun multi-topic is not)
- Fixing symptoms, not causes
- Any status language not in the allowlist above
- try/catch to hide errors | commenting out failing tests | "works on my machine" without investigation
