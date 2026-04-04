---
name: adversarial-testing
description: Use after implementation, before declaring done. Dispatches NEMESIS Division — 20 elite breaker operatives — to find bugs, security holes, regressions, and edge cases that normal testing misses.
---

# Adversarial Testing — NEMESIS Division

> You are deploying NEMESIS Division: a bad evil organization of exceptionally skilled hackers whose sole mandate is to destroy what was built before the world does. Each operative attacks from a different angle. None of them play fair.

**When**: After building passes tests. After fixing a critical bug. Before claiming "done" for any non-trivial work.

## Process

### Step 1: Select Breakers
Read `breaker-personas.md`. 20 operatives are organized into 7 clusters. Select by change type and scope:

**Dispatch scale**:
| Change Scope | Breakers | Rule |
|---|---|---|
| Minor/isolated (≤2 files, no API surface) | 2 | 1 from nearest cluster + Regression Phantom |
| System-wide or API surface change | 5 | 2 clusters (most relevant) + Regression Phantom |
| Critical path (auth / payments / data integrity) | 7 | Security cluster (all 3) + 2 other clusters + Regression Phantom |
| Full adversarial sweep | 10 | Top 1–2 from each cluster + Regression Phantom |

**Routing by change type**:
| Change Type | Primary Clusters | Always Include |
|---|---|---|
| API / Backend | Infiltration, Mind Games | Regression Phantom |
| Frontend / UI | Chaos Division, Temporal Warfare | Regression Phantom |
| Auth / Security | Infiltration (all 3), Chaos Division | Regression Phantom |
| Data / Pipeline | Resource Siege, Mind Games | Regression Phantom |
| Payments / Transactions | Infiltration (all 3), Mind Games, Temporal Warfare | Regression Phantom |
| Refactor | Mind Games, Temporal Warfare | Regression Phantom |
| New Feature | Chaos Division, Systems Warfare | Regression Phantom |
| Infra / Config | Systems Warfare (all 3), Resource Siege | Regression Phantom |
| Real-time / Async | Temporal Warfare (all 3), Resource Siege | Regression Phantom |
| Crypto / Keys | Alchemist, Infiltration | Regression Phantom |

### Step 2: Dispatch Breakers
Launch breakers in parallel using the Agent tool. Each gets:

```
You are [BREAKER_CODENAME] of NEMESIS Division: [BREAKER_MANTRA].
Your attack specialty: [ATTACK_SPECIALTY]

Target: [WHAT_CHANGED — files, functions, endpoints]
Code: [RELEVANT_CODE — extracted inline, max 50 lines]

Use your Signature Moves. Be creative. Go beyond them if you see a new angle.

Return ONLY this block — no prose outside it:
## Findings
- [SEVERITY: critical|high|medium|low] [description]
  Reproduce: [exact input/sequence] → expected [X] → got [Y]

## Tests attempted (required — min 3 before SOLID):
- [specific test or input tried]
- [specific test or input tried]
- [specific test or input tried]

## Attack vector: [SURFACE] — [entry point used, one line]
## Blast radius: CONTAINED | LATERAL | SYSTEMIC
## Rebreak: NEEDED | CLEAN
## Verdict: BROKEN | FRAGILE | SOLID
[MAX 150 words]
```

**Validation**: SOLID without ≥3 Tests attempted = invalid — treat as FRAGILE. SOLID without Blast radius = treat as FRAGILE.

### Step 3: Triage
Collect all findings. Triage by severity:
- **Critical/High** → MUST fix before proceeding. Return to `building`.
- **Medium** → Fix if quick (<5 min), otherwise document as known limitation.
- **Low** → Document only. Do not fix unless trivial.

Use Blast radius to scope the fix:
- `CONTAINED` → fix the specific location
- `LATERAL` → check adjacent code paths before fixing
- `SYSTEMIC` → fix at the pattern level, not just the symptom

### Step 4: Fix-and-Rebreak Cycle
For each critical/high finding:
1. Fix the issue
2. Re-dispatch the SAME breaker with `Delta: [file:lines changed — what was fixed and why]` injected into the prompt
3. Confirm `Rebreak: CLEAN` or verdict changes to SOLID/FRAGILE
4. If still BROKEN → fix is incomplete. Try again (max 3 attempts per operative).

### Step 5: Final Verdict
All dispatched operatives report SOLID or FRAGILE (medium issues documented):
→ Proceed to `verification`

Any operative still BROKEN after 3 attempts:
→ Escalate to user. Do not silently proceed.

**Escalation report** (required format):
```
## NEMESIS Escalation
Operative: [codename]
Finding: [what is broken]
Attempts: 3
  Attempt 1: [what was tried] → still BROKEN — [reason]
  Attempt 2: [what was tried] → still BROKEN — [reason]
  Attempt 3: [what was tried] → still BROKEN — [reason]
Blocked by: [root cause — why all fixes failed]
Needs: [operative's recommendation for unblocking]
```

## Anti-Patterns
- Skipping adversarial testing for "simple" changes (simple changes break things too)
- Dispatching only 1 breaker when routing table calls for a cluster
- Ignoring medium findings without documenting them
- Treating FRAGILE as acceptable without written acknowledgment
- Fabricating operative responses instead of actually dispatching agents
