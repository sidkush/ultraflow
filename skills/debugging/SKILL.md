---
name: debugging
description: Use when encountering any bug, test failure, or unexpected behavior. Systematic 4-phase approach with session scoping, TDD bug fix loop, proof-of-fix gates, blind skeptic verification, and mandatory debug journals.
---

# Systematic Debugging

## Iron Laws
1. **NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**
2. **ONE BUG AT A TIME. NEVER BATCH.**
3. **NO "SHOULD FIX IT" — PROVE IT FIXED.**

## Session Scope Protocol — BEFORE ANYTHING ELSE

**Hard limit: 3 bugs per session.** If more than 3 bugs exist, triage to pick the 3 highest-impact. Remaining go to a follow-up session.

```
Session scope:
  Bug 1: [description] — Status: [QUEUED | ACTIVE | RESOLVED | ESCALATED]
  Bug 2: [description] — Status: QUEUED
  Bug 3: [description] — Status: QUEUED
```

**Sequential Completion Rule:** Bug N+1 cannot start until Bug N reaches RESOLVED or ESCALATED. No interleaving. No "quick detour." If fixing Bug 1 reveals Bug 4, log it but do NOT context-switch — finish Bug 1 first, then re-triage the queue.

**Why:** Sessions with 13+ bug fixes consistently degrade — early fixes succeed, later fixes get buggy as context fills. Shorter focused sessions maintain fix quality throughout.

## UFSD Read Protocol
Before Phase 1:
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
| 4 | Claiming fix without proof | [which fix could be falsely reported as done] |
| 5 | Context degradation from batching | [which bugs could suffer from late-session fatigue] |

**Inject top 2 failure modes into Skeptic dispatch prompt verbatim** under heading "Watch for these failure modes."

## Entry — Choose Path First

| Signal | Path |
|--------|------|
| Obvious typo/import, 1 file, cause immediately visible | **Simple** → Phase 1 + Phase 4 only |
| Cascading errors, 2+ files, ambiguous cause, CI/env mismatch, 2+ prior fix sessions | **Complex** → All phases below |

**Auto-reclassify to Complex if:** fix requires >1 file / fails unexpectedly / error recurs after fix.
**On reclassify (Simple→Complex):** carry Phase 1 artifact verbatim into the new Complex-path Phase 1. Do not restart — existing evidence is still valid.
**Resume (prior attempts exist):** audit prior fix attempts into Phase 1 Evidence Artifact before starting — treat each as a "DENIED" hypothesis with evidence. Do not re-cover already-disproven ground.

---

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

---

## Phase 4: TDD Bug Fix Loop

Only after root cause confirmed, Pattern Exhaustion complete, and Ghost β complete.

**This is the most critical phase. Follow each step exactly. No shortcuts.**

### Step 1: Write Failing Test (RED)
Write a test that **reproduces the exact bug**. This test encodes what "fixed" means.
```
Test file: [path]
Test name: [descriptive name that includes the bug behavior]
```

### Step 2: Confirm Failure
Run the test. It **MUST fail**. If it passes without any code change, your test is wrong — it doesn't actually reproduce the bug. Rewrite it.
```
RED confirmation:
  Command: [exact test command]
  Output: [verbatim failure output]
  Exit code: [non-zero]
```
**Cannot proceed to Step 3 without RED confirmation.**

### Step 3: Implement Minimal Fix
Write the **smallest possible change** that addresses the root cause. Do not refactor. Do not improve adjacent code. Do not add features. Fix the bug and nothing else.

### Step 4: Confirm Pass (GREEN)
Run the same test from Step 1. It **MUST pass**.
```
GREEN confirmation:
  Command: [same test command as Step 2]
  Output: [verbatim pass output]
  Exit code: 0
```
**If it still fails:** analyze why, adjust the fix (not the test), and re-run. Iterate Steps 3-4 until green. Do NOT move on while the test is red.

### Step 5: Regression Check
Run the **full test suite**. Every pre-existing test must still pass.
```
Regression check:
  Command: [full suite command]
  Output: [summary line — e.g., "119 passed, 0 failed"]
  Exit code: 0
```
**If any regression:** fix it before proceeding. The regression fix follows the same TDD loop (write test for regression → confirm red → fix → confirm green). Do NOT skip this.

### Step 6: Proof-of-Fix Gate

**BANNED PHRASES** — using any of these means you have NOT verified:
- "This should fix it"
- "This should work now"
- "I believe this resolves..."
- "The fix looks correct"
- Any variant of "should", "probably", "likely", "I think"

**Required proof artifact:**
```
Proof-of-Fix:
  Pre-fix behavior:  [command] → [failure output verbatim]
  Post-fix behavior: [command] → [success output verbatim]
  Test proof:        [test command] → [pass output verbatim]
  Suite proof:       [suite command] → [N passed, 0 failed]
  Counterexample:    [input that would have triggered the original bug, now caught by the new test]
```
All 5 fields required. Missing any = cannot proceed to Skeptic.

**Cannot produce counterexample →** write `"POSITIONAL FIX — escalate before closing"`.

### Step 7: Pre-Skeptic Confidence
```
Fix depth [1-5]:  1=symptom patched only  5=root invariant enforced at all boundaries
Paths covered [1-5]:  1=original repro only  5=all Pattern Exhaustion locations confirmed fixed
Session penalty: −1 per prior failed session on this bug (sessions so far: N)
Score = depth + paths − penalty  [min 1]
```
Score ≤4 → write: "Causal justification: [why this fix is complete despite low score]" before dispatching.

### Step 8: Skeptic Dispatch
Dispatch `skeptic` subagent **once per session** (see `skeptic.md`).
Pass: Phase 1 Evidence Artifact **verbatim** + Proof-of-Fix artifact + Coverage matrix + environment context + top 2 failure modes from Meta-Cognitive Failure Mode Map. Never pass fix implementation details.
**Intermittent bugs:** note in dispatch — Skeptic runs repro steps 3x and reports all outcomes.

**Status before Skeptic sign-off — only these phrases permitted:**
- `"Awaiting Skeptic sign-off"`
- `"Fix session closed — Skeptic dispatched"`
- `"Skeptic returned [verdict] — [next action]"`

If Skeptic → `BUG NOT DETERMINABLE`: improve Phase 1 artifact and re-dispatch once.
If `BUG NOT DETERMINABLE` twice consecutive → STOP, escalate to user.

---

## Phase 5: Debug Journal

**After Skeptic sign-off (RESOLVED) or escalation (ESCALATED), write a debug journal.**

Save to: `docs/ultraflow/debug-journals/[YYYY-MM-DD]-[slug].md`
Use template from `debugging/debug-journal-template.md`.

The journal captures the **full story** of the bug — not just what changed, but why it existed, how it got there, and how to prevent it. These journals are consumed by `/graphify` and future `/init` sessions to build institutional memory.

**Journal is mandatory.** No bug is RESOLVED without its journal committed alongside the fix.

### What makes a good journal:
- **Root cause chain** — trace back to the original decision or commit that introduced the bug
- **"How did we miss it"** — what test/review/process gap allowed this bug to ship
- **Prevention rule** — a concrete, actionable rule (not "be more careful")
- **Pattern tag** — categorize the bug type so future sessions can search for similar patterns

---

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
Journal: [link to debug-journals/[file].md]
```

## Session Completion
After all bugs in the session scope are RESOLVED or ESCALATED:
1. Verify session scope — all bugs accounted for
2. All debug journals committed
3. UFSD updated
4. Print session summary:
```
Session complete:
  Bug 1: [description] — RESOLVED (journal: [link])
  Bug 2: [description] — RESOLVED (journal: [link])
  Bug 3: [description] — ESCALATED (reason: [X])
  Bugs deferred to next session: [list or "none"]
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
- Retrospective confirmation — writing CONFIRMED without a prior Prediction block
- Accepting a hypothesis without completing the Counterfactual Gate
- Labeling a SYMPTOM-level hypothesis as ROOT_CAUSE
- "Let me try this..." without evidence (shotgun debugging)
- Unrelated changes in one Fix Session (related atomic edits are correct — shotgun multi-topic is not)
- Fixing symptoms, not causes
- **Working on Bug N+1 before Bug N is RESOLVED** — sequential completion violation
- **Saying "should fix" / "probably works" / "looks correct"** — proof-of-fix violation
- **Skipping the RED confirmation** — writing a fix before confirming the test actually fails
- **Batching multiple bug fixes** — one at a time, always
- Any status language not in the allowlist above
- try/catch to hide errors | commenting out failing tests | "works on my machine" without investigation
- Skipping UFSD Read at session start when a UFSD file exists
- Leaving unvalidated ASSUMPTIONs without logging them as risk items in UFSD Write
- **Skipping the debug journal** — no bug is closed without its journal
