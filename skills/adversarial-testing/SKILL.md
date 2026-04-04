---
name: adversarial-testing
description: Use after implementation, before declaring done. Dispatches ALL 20 NEMESIS Division operatives in parallel to find bugs, security holes, regressions, and edge cases that normal testing misses. Zero routing shortcuts, zero operative sampling. The full division is the product.
---

# Adversarial Testing — NEMESIS Division

> You are deploying NEMESIS Division: a bad evil organization of exceptionally skilled hackers whose sole mandate is to destroy what was built before the world does. Each operative attacks from a different angle. None of them play fair.

## Core Principle
Twenty operatives. Every run. Always. No sampling, no "only 5 for simple changes." NEMESIS operates at full strength or not at all. The collective destruction of all 20 attack surfaces — that is the division.

**When**: After building passes tests. After fixing a critical bug. Before claiming "done" for any non-trivial work.

## Process

### Step 1: Extract Target
Before dispatch, distill the target:
```
Target: [≤100 words — what was built/changed]
Attack surface: [entry points, data flows, auth boundaries, external integrations]
Change scope: [files modified, APIs touched, data schemas altered]
Known weak points: [any areas already suspected of fragility]
```

### Step 2: Dispatch All 20 Operatives Simultaneously
Launch **all 20 NEMESIS operatives** using the Agent tool — in a single parallel dispatch. All 20 must return before triage begins.

Read `breaker-personas.md` for full operative profiles. The 20 operatives (dispatch in this order, all at once):

**Cluster I — Infiltration**: 1 Phantom Injector, 2 Ghost in the Auth, 3 Baron Von CSRF
**Cluster II — Chaos Division**: 4 Professor Overflow, 5 Sigil Wraith, 6 Architect Void
**Cluster III — Temporal Warfare**: 7 Phantom Interval, 8 Null Epoch, 9 Vector Lace
**Cluster IV — Resource Siege**: 10 Lethe, 11 Sisyphus, 12 Tantalus
**Cluster V — Systems Warfare**: 13 Seraphex, 14 Voltgrieve, 15 Malvareth
**Cluster VI — Mind Games**: 16 Paradox, 17 Ouroboros, 18 Meridian
**Cluster VII — Deep Specialists**: 19 The Alchemist, 20 Regression Phantom

Each operative prompt:
```
You are [OPERATIVE_CODENAME] of NEMESIS Division: [OPERATIVE_MANTRA].
Your attack specialty: [ATTACK_SPECIALTY from breaker-personas.md]
Your signature moves: [4 SIGNATURE_MOVES from breaker-personas.md]

Target: [TARGET from Step 1]
Attack surface: [ATTACK_SURFACE from Step 1]
Code under attack: [RELEVANT_CODE — max 60 lines, inline]

Use your Signature Moves. Be creative. Go beyond them if you see a new angle. Your job is to break this — not validate it.

Return ONLY this block — no prose outside it:
## Findings
- [SEVERITY: critical|high|medium|low] [exact description — what breaks, not what might break]
  Reproduce: [exact input/sequence/state] → expected [X] → got [Y]
  (No exact reproduce step = finding is INVALID)

## Tests attempted (min 5 before SOLID):
- [specific input/state tried and exact observed result]
- [specific input/state tried and exact observed result]
- [specific input/state tried and exact observed result]
- [specific input/state tried and exact observed result]
- [specific input/state tried and exact observed result]

## Attack vector: [SURFACE] — [exact entry point used, one line]
## Blast radius: CONTAINED | LATERAL | SYSTEMIC
## Rebreak: NEEDED | CLEAN
## Verdict: BROKEN | FRAGILE | SOLID
[MAX 200 words]
```

**Anti-hallucination validation gate** (hard — blocks if violated):
- Findings missing exact Reproduce step → **INVALID finding** — discard that finding only, keep others
- Tests attempted < 5 → SOLID verdict overridden to **FRAGILE** automatically
- Tests attempted that are generic ("tested with bad input", "tried edge cases") → **INVALID test** — does not count toward the 5 minimum
- SOLID without Blast radius → treated as **FRAGILE**
- SOLID without Attack vector → treated as **FRAGILE**
- Operative returns narrative prose instead of block format → **FABRICATED** — treat as not dispatched, re-dispatch once
- Critical finding with no severity label → labeled **high** automatically

**Partial failure**: If any operatives fail/error → re-dispatch those slots once. If re-dispatch also fails → proceed without them, note count. If fewer than 12 valid operative reports remain → HALT, tell user how many succeeded, ask whether to proceed or re-run.

### Step 3: Mega-Triage — Priority Matrix

After all 20 operatives return, do NOT present 20 individual reports. Instead:

**3a. Severity × Blast Radius Priority Matrix**:
```
              CONTAINED   LATERAL   SYSTEMIC
critical  →     P1          P0        P0
high      →     P2          P1        P1
medium    →     P3          P2        P2
low       →     P4          P3        P3
```
P0 = Stop everything. Fix before any other work.
P1 = Fix before proceeding. Rebreak required.
P2 = Fix if <10 min, otherwise document as known limitation.
P3 = Document only.
P4 = Note for backlog.

**3b. Triage Table**:
```markdown
| Priority | Operative | Severity | Blast Radius | Finding (1 line) | Reproduce |
|----------|-----------|----------|--------------|-----------------|-----------|
| P0       | [name]    | critical | SYSTEMIC     | [description]   | [input→output] |
| P1       | [name]    | high     | LATERAL      | [description]   | [input→output] |
...
```
Sort by priority (P0 first). Include all BROKEN and FRAGILE findings.

**3c. Unanimous Weaknesses** (≥15/20 operatives flag the same class of vulnerability):
→ List each. These are structural — any approach chosen must address them before closing.

**3d. Strong Attack Signals** (≥10/20 operatives independently discover the same attack surface):
→ Mark with ⚠️. If 2+ attack surfaces each have ≥10 votes, escalate to user before fixing — indicates architectural issue, not bug.

**3e. Minority Discoveries** (found by <3 operatives but with critical or high severity):
→ List with operative name. Do not discard — rare discoveries are often the most exploitable.

**3f. Clean Operatives**: List operatives who returned SOLID. These are the confirmed-safe attack surfaces.

### Step 4: Fix-and-Rebreak Cycle

**Rebreak scope is determined by Blast Radius**:
- `CONTAINED` → re-dispatch only the operative who found the issue
- `LATERAL` → re-dispatch the full cluster the operative belongs to
- `SYSTEMIC` → re-dispatch ALL 20 operatives (full division sweep)

For each P0/P1 finding:
1. Fix the issue
2. Re-dispatch per scope above with `Delta: [file:lines changed — what was fixed and why]` injected into each prompt
3. Confirm `Rebreak: CLEAN` or verdict changes to SOLID/FRAGILE
4. If still BROKEN → fix is incomplete. Try again (max 3 attempts per operative).

For P2 findings: Fix → single operative rebreak → confirm CLEAN or document as known limitation.

### Step 5: Final Verdict

All dispatched operatives report SOLID or FRAGILE (P2/P3/P4 issues documented):
→ Proceed to `verification`

Any operative still BROKEN after 3 attempts:
→ Escalate to user. Do not silently proceed.

**Escalation report** (required format):
```
## NEMESIS Escalation
Operative: [codename]  Cluster: [cluster name]
Finding: [what is broken]
Priority: [P0/P1]  Blast radius: [CONTAINED|LATERAL|SYSTEMIC]
Attempts: 3
  Attempt 1: [what was tried] → still BROKEN — [reason]
  Attempt 2: [what was tried] → still BROKEN — [reason]
  Attempt 3: [what was tried] → still BROKEN — [reason]
Blocked by: [root cause — why all fixes failed]
Needs: [operative's recommendation for unblocking]
```

**SYSTEMIC finding still BROKEN after 3 attempts**: Do not proceed under any circumstances. This is an architectural failure — escalate to `ultraflow:council` with the finding as the problem statement.

## Anti-Patterns
- Dispatching fewer than 20 operatives because the change "seems simple"
- Fabricating operative responses instead of actually dispatching agents
- Treating FRAGILE as acceptable without written acknowledgment in triage table
- Skipping SYSTEMIC rebreak (full division sweep) — SYSTEMIC means the whole system is affected
- Accepting "tested with bad input" as a valid test (must be specific: exact value, exact sequence)
- Letting P0 findings sit while fixing P2/P3 findings first
- Proceeding to verification with any operative still BROKEN
- Counting INVALID findings toward the fix count (each INVALID = zero coverage)
