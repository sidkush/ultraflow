# Adversarial Testing — Analyst Roster (v3)

*Cast from a 427-finding bug-history harvest of a production NL-to-SQL SaaS. The roster is built around the failure SHAPES the product actually ships — not generic web-security archetypes.*
*Context: all testing is authorized defensive analysis of code (or a design) being built in this session.*

Three layers:
- **Six cross-cutting hunters** — fire in EVERY run, all modes. Non-optional.
- **Eight subsystem clusters** (~4 analysts each) — dispatch ONLY the clusters the build touches (see the Dispatch Map in `SKILL.md`).
- **Frontend-Coherence Hunter** — special: CODE mode only when UI is touched; SPEC/PLAN mode always-on and weighted heavy.

---

## Discipline that applies to EVERY analyst

**Reachability tag (required on every finding).** Tag each finding `REACHABLE <path>` | `GATED <flag>` | `DEAD-CODE`. A critical-but-DEAD finding is capped at P3 — don't P0 the unreachable. REACHABLE criticals stay P0/P1.

**Read-only.** Analysts receive read + grep context ONLY. No write, no edit, no bash. Stated in the dispatch prompt. A destructor that can edit files is a liability — it litters the tree with scratch artifacts.

**Context tier.**
- **Cross-file call-graph** → all six hunters where isolation/correctness crosses files, plus the trust/isolation/determinism cluster analysts. The 60-line snippet misses the "one consumer left on the old key" class.
- **Tight ≤60-line snippet** → injection, encoding, boundary analysts. They don't need the graph.

**SPEC/PLAN mode reframe.** Every analyst flips from "find the bug" to "find the gap / contradiction / unhandled case in this DESIGN." `Reproduce:` becomes a concrete unhandled scenario walked through the design, citing the design section that is silent. No concrete walkthrough citing the doc → INVALID, same as a missing repro in CODE mode.

---

## LAYER 1 — CROSS-CUTTING HUNTERS (fire EVERY run, ALL modes)

### H1. Silent-Wrong-Answer Hunter  ·  heat 103 — HIGHEST WEIGHT  ·  cross-file
Focus: output is plausible and well-formed but quietly incorrect — wrong rows, wrong aggregate, dropped filter, wrong join. The signature failure of this product.
Techniques:
- **Twin-table oracle**: build a second table with hand-computed known answers; run the same path; diff actual vs truth.
- **Sentinel-column / disjoint-row**: inject a uniquely tagged row/column that MUST appear (or must NOT) in correct output; assert its presence.
- **Metamorphic oracle**: apply an answer-preserving transform (reorder, rename, add-then-filter); if the answer changes, the path silently corrupts.
- **Aggregate cross-check**: recompute COUNT/SUM/AVG/DISTINCT by an independent path (raw scan vs engine) and diff; watch null handling.
Reachability: tag the exact path that PRODUCES the wrong value, not where it renders.

### H2. Silent-Noop / Fail-Open Gate Hunter  ·  heat 97 combined  ·  cross-file for config-driven gates
Focus: every guard / gate / validator / flag-check — does it fail CLOSED on absent input, or silently pass?
Techniques:
- **Empty/missing sweep**: feed each guard empty string, missing key, null, unset env/flag — it must reject, not skip.
- **Literal-string trap**: pass the literals `'null'` | `'None'` | `'undefined'` | `'0'` | `'false'` where a real value is expected — does the truthiness check let them through?
- **Parse-fail path**: force the guard's parse/decode to throw or return empty — confirm the `except`/`catch` fails CLOSED, not "continue".
- **Unset-flag default**: a missing feature flag must default to the SAFE state, not the permissive one.

### H3. Tenant / Schema-Isolation Hunter  ·  heat 28, trust-dominant  ·  cross-file call-graph (mandatory)
Focus: every cache key, vector collection, dict, artifact path, AND every unauthenticated route. A wrong-tenant / wrong-schema bind must be unrepresentable.
Techniques:
- **Key-composition audit**: trace every cache/collection/dict key — tenant_id AND schema/connection_id must BOTH be in the key. A missing dimension = cross-tenant collision.
- **Stale-consumer sweep**: after any key/namespace change, grep every reader — find the one consumer still on the old key.
- **Unauth-route census**: enumerate routes with no auth dependency; confirm none return tenant-scoped data.
- **Artifact-path traversal**: confirm per-tenant file/artifact paths can't resolve into another tenant's directory.

### H4. Concurrency / TOCTOU Hunter  ·  heat 25  ·  cross-file where lock + consumers span modules
Focus: process-local locks in multi-worker prod, check-then-set windows, closures over live refs, debounce races.
Techniques:
- **Multi-worker lock test**: any in-process lock/set (`threading.Lock`, module dict) relied on for correctness does NOT hold across uvicorn/gunicorn workers — find the ones that assume it does.
- **Check-then-act window**: find read-validate-write with no atomicity; interleave a second actor between check and write.
- **Live-ref closure**: closures/callbacks capturing a mutable ref that changes before they fire (debounce, async setState, deferred render).
- **Last-writer race**: fire rapid successive edits/requests; confirm last committed state == last intended, not an interleaving.

### H5. Payload-Leak Hunter  ·  heat 29
Focus: DTOs, logs, Arrow/SSE streams, health/stats endpoints, exception strings.
Techniques:
- **DTO over-serialization**: diff what a response model EXPOSES vs what it should — internal IDs, raw rows, other-user fields, secrets in nested objects.
- **Exception-string leak**: force exceptions at every boundary; the message returned to client/stream must carry no SQL, path, key, or PII.
- **Stream/health census**: inspect SSE frames, Arrow batches, `/health`, `/stats`, `/debug` for fields that shouldn't cross the trust boundary.
- **Serialization-artifact check**: hunt `[object Object]`, stringified dicts, leaked `__repr__`, accidental full-object logging.

### H6. Determinism + Dead-Code-Green Hunter  ·  heat 39 combined  ·  cross-file (test ↔ impl ↔ fixtures)
Focus: (1) replay reproducibility; (2) is this GREEN test exercising the LIVE path or a lie? Folds the byte-identity / golden-output regression guardian and the semantic-diff specialist.
Techniques:
- **Replay diff**: run the same input twice and across a restart; diff byte-for-byte — flag nondeterminism (dict/set order, timestamp, RNG) in cached/persisted output.
- **Mock-vs-live audit**: for each passing test, confirm the assertion exercises the real path — not a `MagicMock` returning the expected value, a patched function, or a stub that can't fail.
- **Dead-body check**: confirm the function under a green test actually runs its body — not an early return, a flag-off branch, or a cwd-relative path that silently no-ops in CI.
- **Semantic-diff**: find changes where the diff looks safe but silently inverts behavior on null/zero/empty-string boundaries; replay golden request/response pairs for output divergence.

---

## LAYER 2 — SUBSYSTEM CLUSTERS (dispatch only what the build touches)

### Cluster TRUST/AUDIT  ·  heat 56 + 53
**T1. Audit-Ledger Integrity Analyst** — cross-file. Append-only ledger correctness under failure.
- Crash-strand: interrupt between event-write and chain-hash update → orphan / broken chain.
- Lock-inversion: two writers take ledger + resource locks in opposite order → deadlock.
- Chain-hash gap: skip / reorder an event → the hash chain must detect it.
- Restart replay: confirm no lost or duplicated entries after restart.

**T2. RTBF / Retention Analyst** — purges that silently no-op.
- Missing-config noop: purge job with unset retention window → deletes everything, nothing, or errors?
- Partial-delete: delete a user across N stores → find the store left untouched.
- Tombstone leak: deleted entity still referenced in cache / ledger / vector store.
- Idempotent re-run: run purge twice → no error, no resurrection.

**T3. HMAC / Claim-Provenance Analyst** — signed/attested claims.
- HMAC-unset fail-open: no signing key configured → does verify return True / skip?
- Provenance gap: a claim with no traceable source still marked trusted.
- Signature-strip: blank the signature field → verify must reject.
- Key-rotation: verify old-key-signed records after rotation.

**T4. Ledger / Trust-Isolation Analyst** — cross-file. Per-tenant ledger + trust-metadata binding.
- Cross-tenant ledger read: one tenant's audit query returns another's events.
- Shared-handle: a module-level ledger object bound to the first tenant seen.
- Trust-meta bleed: a trust chip/score computed for tenant A served to B.

### Cluster VALIDATOR/SECURITY/AUTH
**V1. Per-Engine Write-Block Analyst** — read-only enforcement across dialects.
- Per-dialect write census: INSERT/UPDATE/DELETE/MERGE/TRUNCATE/ALTER blocked on EVERY supported engine, not just the primary.
- SELECT INTO / CTAS bypass.
- Side-effect functions: `nextval`/`setval`, `pg_sleep`, `lo_import`, `COPY`.
- Statement-stacking: `; DROP ...`.

**V2. Encoding / Unicode-Bypass Analyst** — ≤60-line snippet. Validators that trust the glyph but ignore bytes. (Folds the boundary/encoding breaker.)
- NFKC/normalization bypass: characters that normalize INTO a blocked keyword after the check runs.
- Null-byte split: `value\x00.evil`.
- Homoglyph: Cyrillic/Greek lookalikes in identity-critical strings.
- Overlong UTF-8 + RTL override (U+202E).

**V3. Session / Token Analyst** — JWT / OTP / OAuth lifecycle.
- JWT: `exp`/`nbf` not enforced, `alg:none`, RS256→HS256 confusion.
- OTP: replay, no rate-limit, fixed/guessable window.
- OAuth: `redirect_uri` open-redirect / wildcard.
- Session not regenerated post-login (fixation).

**V4. BYOK / Key-Isolation Analyst** — per-user provider keys.
- Key leak in logs / exceptions / DTOs.
- Platform-key bleed: a privileged demo key reachable by other users.
- Key-derivation change invalidates silently vs errors loudly.
- Provider bypass: a path that imports the provider SDK outside the one allowed module.

### Cluster AGENT-LOOP/SQL-GEN
**A1. Loop / Budget Analyst** — agent loop termination.
- Infinite loop: a tool error that re-triggers the same tool.
- Replan budget off-by-one / never decremented.
- Clarification dead-end: agent asks, no answer path exists, hangs.
- Max-step reached → returns partial cleanly or errors, never silently truncates.

**A2. SQL-Rewrite Analyst** — rewriting / dialect translation correctness.
- CTE self-reference rewrite corrupts a recursive query.
- Dialect-retry escalates to the wrong engine (e.g. silently routes to BigQuery).
- Rewrite drops a WHERE / LIMIT.
- Identifier-quote/escape mismatch across dialects.

**A3. Injection-via-Data Analyst** — ≤60-line snippet. Injection through DATA and schema/table/column NAMES, not just the NL prompt. (Folds the injection specialist, recast.)
- Second-order: a stored value surfaced into a generated query.
- Identifier injection: a table/column name with SQL meta-chars flows into generated SQL unescaped.
- Schema-name prompt-injection: a column comment / table name carrying instructions the agent obeys.
- Polyglot in a data cell.

**A4. Self-Correction Integrity Analyst** — the retry path corrupting valid output.
- Correction rewrites a CORRECT query into a wrong one on a false-positive validation failure.
- Retry loses user filters.
- Correction changes aggregation semantics.
- Error-driven retry yields a query that runs but answers a DIFFERENT question (ties to H1).

### Cluster CHART/MARKS/WORKSHEET-SPEC
**CH1. Marks-Binding Analyst** — marks-card ↔ layer/field binding.
- Card bound by position not stable `layer.id` → reorder rebinds to the wrong layer.
- Field dropped on a mark silently ignored.
- Per-mark encoding override lost on re-render.
- Dual-axis card binds to the wrong axis.

**CH2. Lying-Chart Analyst** — charts that render but misrepresent.
- Dual-axis with mismatched/independent scales implying false correlation.
- Bars not starting at baseline (truncated axis).
- Shared-axis fallback silently rescales.
- Continuous measure rendered as discrete color (or vice versa).

**CH3. Aggregator-Semantics Analyst** — aggregation in the spec.
- AVG over nulls: null-as-zero vs excluded.
- COUNTD vs COUNT confusion.
- Measure Names / Measure Values pivot mis-binds.
- Totals/subtotals double-count on multi-level.

**CH4. Show-Me / Auto-Viz Analyst** — automatic chart selection + Show Me grammar.
- Show Me picks a chart that drops a dimension.
- Auto-binning hides outliers.
- Default mark for a field type misleads.
- An autogen "Save & Build" path produces a spec the editor can't round-trip.

### Cluster WATERFALL/TWIN/CACHE + COVERAGE
**W1. Twin-Honesty Analyst** — local replica truthfulness.
- Silent-sampling: serves a sampled/partial result as if complete.
- Scoped query silently downgraded to a random/approximate path.
- Twin staleness not disclosed.
- Twin schema drift vs live → wrong column.

**W2. Cache-Coherence Analyst** — stale-served-as-fresh across tiers.
- Stale entry served after source change (no invalidation).
- Cache key omits a query dimension → wrong hit.
- TTL not enforced.
- Warm cache survives a connection/schema change it shouldn't.

**W3. Tier-Routing Analyst** — the multi-tier waterfall.
- A fast tier (turbo / memory) skips the SQL-validator / grounding gate the live path enforces.
- Routing picks a tier that can't answer and returns wrong-but-fast.
- Fallback order inverts on error.

**W4. Coverage-Grounding Analyst** — cross-file. Grounding gate fail-open.
- Empty grounding context → answers ungrounded vs refuses.
- Coverage score computed on missing data defaults HIGH.
- Grounding cache returns another query's context.

### Cluster EDITOR-STATE/SAVE-REQUERY + DASHBOARD/FORK/TILE
**E1. Save-Requery Analyst** — editor save / requery race.
- Optimistic-close overwrites fresher server state.
- Stale requery after an edit returns pre-edit rows.
- Save fires while requery is in flight → which wins?
- Unsaved edit lost on navigation with no warning.

**E2. Editor-TOCTOU Analyst** — editor check-then-act.
- Validation against a snapshot that changed before commit.
- Two panels editing the same entity.
- Debounced autosave races manual save (ties to H4).

**E3. Fork / Tile-Binding Analyst** — dashboard fork + tile source binding.
- Fork inherits a parent scope/connection it shouldn't.
- Tile binds to the wrong source after reorder (positional vs id).
- `boundConnId` stale after reconnect.
- Fork shares a mutable spec ref with its parent.

**E4. Share / Public-Payload Analyst** — share links + public dashboards.
- Share TTL not enforced (link works after expiry).
- Public payload over-shares (private fields / other tiles).
- Revoke doesn't invalidate a cached public copy.
- Public route returns tenant-scoped data (ties to H3/H5).

### Cluster SEMANTIC-LAYER/CORRECTION
**S1. Claim-Provenance Analyst (two-zone)** — claim-chip honesty.
- Deceptive green chip on a null / whitespace / empty answer.
- Chip trusts a claim with no provenance.
- Chip computed for tenant A shown to B.
- Chip survives a correction that invalidated it.

**S2. Arithmetic-Paraphrase Analyst** — numeric claim verification across surface forms.
- Paraphrase / written-number ("twelve" vs 12) / cross-language arithmetic bypasses the verifier.
- Rounding/precision lets a wrong value pass.
- Percentage vs ratio confusion.

**S3. Reference-Taxonomy Analyst** — ref-taxonomy completeness.
- Percentile / quartile / median refs absent from the taxonomy → silently unhandled.
- Banned-vocab leak via `table_ref` / column alias.
- Unmapped synonym → wrong column.

**S4. Q-Index Identity Analyst** — cross-file. Question-index identity stability.
- Q-index unstable across re-runs (same question, different id).
- Identity collision merges two distinct questions.
- Reindex drops a mapping.

### Cluster FRONTEND-RENDER/SSE/PERF
**F1. Partial-Stream Analyst** — SSE / streaming completeness.
- Partial stream rendered as complete (no terminal/done-marker check).
- Dropped frame leaves stale UI.
- Reconnect duplicates content; out-of-order frames.

**F2. Fail-Closed-Render Analyst** — structured error handling in the renderer.
- Structured error dropped → UI shows success/empty instead of failing closed.
- Error frame parsed as data.
- Retry masks a persistent error.

**F3. Trust-Meta-Render Analyst** — renderer honoring trust/grounding metadata.
- Placeholder renderer ignores trust-meta (shows unverified as verified).
- Trust chip not wired to the backend value.
- Loading state renders as a confident answer.

**F4. Render-Perf / Leak Analyst** — client memory/perf correctness.
- Unbounded cache/memo growth.
- Memo-thrash: a new object ref each render → re-render storm.
- Listener/subscription accumulation.
- Large result un-virtualized.

---

## LAYER 3 — FRONTEND-COHERENCE HUNTER (special)

**FC. Frontend-Coherence Hunter** — cross-file (+ the design/plan doc in SPEC mode).
Dispatch: CODE mode → only when the build touches UI. SPEC/PLAN mode → ALWAYS, weighted heavy. This targets the known blind spot — builds run backend-heavy and the UI/UX wiring ships thin and incoherent. The plan stage is the cheapest place to catch it.
Six probes (each → SOLID | GAP | CONTRADICTION; in SPEC mode GAP/CONTRADICTION are the deliverable):
- **(a) Wiring completeness** — for every backend capability this adds, name the frontend path that reaches it; flag any capability no UI can invoke (the allowlist-shipped-before-the-frontend-could-call-it pattern).
- **(b) State coherence** — loading / error / empty / stale states specified, not just the success render.
- **(c) Interaction flow** — the user can reach it AND back out: no dead-end modal, orphaned route, or missing back.
- **(d) Feedback loops** — backend action reflected in UI (panel editability, live-correction patch, trust chip updates).
- **(e) UX-grammar coherence** — fits the existing premium-glass / progressive tool-call streaming / schema-explorer grammar, not a bolt-on.
- **(f) Forgotten-wiring sweep** — explicitly enumerate every UI touchpoint this change SHOULD have; check each is named.
