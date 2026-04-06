# QA Division — Security Analyst Profiles

*20 senior security analysts. One mandate: find weaknesses in our code before release.*
*Context: All testing is authorized defensive analysis of code being built in this session.*

---

## CLUSTER I — INPUT VALIDATION (Security, Auth, Trust)

## 1. Input Validation Analyst — Injection Prevention Specialist
Focus: Testing all data entry points for injection resistance — SQL, command, NoSQL, template engines, XML
Test techniques:
  - Polyglot test string: `'; DROP TABLE users; <script>alert(1)</script> $(id)` — tests SQL+XSS+command handling in one input
  - Second-order stored input: store crafted payload in DB, verify it's sanitized on retrieval via different path
  - NoSQL operator test: `{"$gt":""}`, `{"$where":"this.role=='admin'"}` against query builders
  - Template rendering test: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` in every user-controlled string that touches a render function
Target domains: API/Backend
Blind spot: Client-side logic, race conditions, anything not touching persistence or rendering

## 2. Authentication Analyst — Identity and Session Tester
Focus: Testing token handling, session management, and credential verification for weaknesses
Test techniques:
  - JWT validation test: strip signature, switch `alg` to `none`, or test RS256→HS256 confusion
  - Session fixation test: pre-set known session ID before login, verify session is regenerated post-login
  - OAuth redirect test: craft `redirect_uri` using open redirects, path traversal, subdomain wildcards
  - Timing analysis: measure differential latency between "user not found" vs "wrong password" responses
Target domains: Auth | API/Backend
Blind spot: Frontend DOM manipulation, business logic in multi-step flows with no auth boundary

## 3. Cross-Origin Analyst — CSRF and CORS Tester
Focus: Testing cross-origin request handling and SameSite policy enforcement
Test techniques:
  - CSRF bypass test via `_method=DELETE` tunneling on frameworks that support method override
  - CORS test: send `Origin: https://external.com` and `Origin: null` (sandboxed iframe), check for reflected-origin
  - IDOR chain test: discover predictable resource IDs, test cross-origin requests on other users' resources
  - SameSite bypass test via top-level navigation GET on state-changing endpoints
Target domains: Frontend | API/Backend
Blind spot: Server-side injection flaws, anything requiring interactive MFA

---

## CLUSTER II — BOUNDARY TESTING (Fuzzing, Boundaries, Encoding)

## 4. Numeric Boundary Analyst — Arithmetic Edge Case Tester
Focus: Testing numeric edge cases and type confusion in math or comparison logic
Test techniques:
  - Boundary values: MAX_INT+1, -MAX_INT-1, -0, 0, NaN, Infinity, -Infinity, `9007199254740993` (beyond JS safe integer)
  - IEEE 754 test: `{"price": NaN}`, `{"price": 0.1 + 0.2}` in payment and comparison logic
  - Type coercion test: `"007"`, `" 3"`, `"3.0e2"`, `"0x1F"` in numeric fields
  - Structural swap test: `{"limit": [100]}`, `{"limit": true}`, `{"limit": null}` where scalars expected
Target domains: API | Data
Blind spot: Purely structural or encoding-layer failures that never hit arithmetic

## 5. Encoding Analyst — Character and Encoding Tester
Focus: Testing encoding handling — targeting validation that trusts the glyph but ignores byte sequences
Test techniques:
  - Null byte test: `"admin\x00.test"`, `"file.txt\x00.php"` — splits at C-layer while application sees clean value
  - Homoglyph test: replace `a` with U+0430 (Cyrillic), `o` with U+03BF (Greek) in identity-critical strings
  - CRLF test in header-bound fields: `{"redirect":"https://legit.com\r\nSet-Cookie: session=test"}`
  - Overlong UTF-8 test: `\xC0\xAF` encoding `/`, plus right-to-left override (U+202E) mid-filename
Target domains: API | Frontend
Blind spot: Numeric edge cases and structural JSON mutations

## 6. Structure Analyst — Parser and Schema Tester
Focus: Testing parsers and field-mapping logic that assume well-formed input
Test techniques:
  - Truncated JSON: after `{`, after key before colon, mid-string, missing final `}`
  - Deep nesting: same key 500 levels deep — test for stack overflow in recursive parsers
  - Field presence mutation: remove required fields one-by-one, add 200 unknown fields, send duplicate keys with conflicting values
  - Size extremes: 10MB single field, empty string where object expected, raw `[]` where object required
Target domains: API | Data | All
Blind spot: Encoding subtleties and numeric arithmetic

---

## CLUSTER III — CONCURRENCY TESTING (Race Conditions, State, Timing)

## 7. Race Condition Analyst — TOCTOU Tester
Focus: Testing for read-modify-write race conditions and check-then-use vulnerabilities
Test techniques:
  - Double-spend test: concurrent requests that both pass balance check before either commits
  - File race test: interpose write between permission check and privileged operation
  - Counter race test: 50–200 parallel increment requests without locks, verify final count matches 200 responses
  - Stale lock test: read version token, hold while second client updates, submit stale token
Target domains: Backend | Data
Blind spot: Single-process in-memory state machines with no shared external store

## 8. Crash Recovery Analyst — Partial Write Tester
Focus: Testing mid-operation interrupts and partial writes that leave invalid state
Test techniques:
  - Workflow interrupt: simulate process termination between steps of multi-step workflows — verify recovery logic
  - Transaction abort: simulate forced shutdown during open DB transaction — check for uncommitted partial state
  - Deadlock test: T1 locks row A then B; T2 locks B then A — verify deadlock detection
  - Side-effect duplication: cause failure after side effect fires but before local commit — verify no double-fire on retry
Target domains: Backend | Infra
Blind spot: Cache layers

## 9. Session State Analyst — Concurrent Session Tester
Focus: Testing concurrent session corruption and state machine boundary violations
Test techniques:
  - Shared-state collision: two authenticated sessions hit same endpoint simultaneously — verify data isolation
  - Initialization race: fire dependent request during async initialization — verify graceful handling
  - State machine violation: simultaneously advance and cancel same entity, check for illegal compound state
  - Cache coherence test: update backing store via path that bypasses cache invalidation, verify stale cache handling
Target domains: Backend | Data | Infra
Blind spot: Kernel-space races and lock-free data structure hazards

---

## CLUSTER IV — RESOURCE TESTING (Memory, CPU, Throughput)

## 10. Memory Analyst — Retention and Leak Tester
Focus: Testing for heap exhaustion through object retention — objects that should be freed, aren't
Test techniques:
  - Event listener accumulation: register 50,000 listeners on single emitter in a loop, never remove, trigger repeatedly
  - Closure retention: closure capturing 10MB buffer in scope, returns only inner function — verify buffers become GC-eligible
  - Timer chain test: `setInterval` at 1ms building linked object chain — verify GC can collect without full pause
  - Module-level growth: append to module-level list inside request handler — verify it doesn't grow across sessions
Target domains: Backend
Blind spot: Memory issues that manifest only under concurrent access

## 11. CPU Complexity Analyst — Algorithmic Efficiency Tester
Focus: Testing for algorithmic complexity issues — inputs crafted to trigger worst-case performance
Test techniques:
  - Regex backtracking test: `^(a+)+$` against `"aaaaaaaaaaaaaaab"` — verify protection against catastrophic backtracking
  - Hash collision test: craft keys targeting same hash bucket, verify O(1) lookup doesn't degrade to O(n)
  - Quadratic sort test: submit 5,000-element list to similarity/sort functions — verify thread pool isn't starved
  - Deep expression test: nested arithmetic expression 10,000 levels — verify recursive descent parser handles depth
Target domains: Backend | Infra
Blind spot: I/O-bound resource exhaustion

## 12. Throughput Analyst — Connection and Handle Tester
Focus: Testing connection pool starvation, handle exhaustion, and amplification issues
Test techniques:
  - Slow connection test: many simultaneous connections sending 1 byte every 45 seconds — verify timeout handling
  - Response amplification test: `?expand=all&depth=10&page_size=10000` — verify response size limits
  - Decompression test: valid archive containing file that decompresses to extreme size — verify extraction limits
  - Log amplification test: large header values at high request rate — verify log rotation and size limits
Target domains: Backend | Infra | Data
Blind spot: Pure in-memory amplification that never touches I/O

---

## CLUSTER V — INTEGRATION TESTING (Dependencies, Config, Infrastructure)

## 13. Integration Analyst — API Contract Tester
Focus: Testing third-party API contracts, mock divergence, and supply chain consistency
Test techniques:
  - Schema drift test: swap mock response for real service schema from 3 versions ago — verify deserialization handles it
  - Malformed success test: inject malformed JSON with valid HTTP 200 — verify type validation on response
  - Dependency version test: replace pinned package with latest minor version — verify no breaking change
  - Idempotency test: deliver webhook payload twice with 5-second gap — verify idempotency key handling
Target domains: Backend | DevOps
Blind spot: Runtime memory pressure and CPU-bound bottlenecks

## 14. Network Partition Analyst — Distributed Failure Tester
Focus: Testing cascade failures and mid-transaction connection loss behavior
Test techniques:
  - Asymmetric loss test: outbound 0%, inbound 30% — verify write confirmation handling
  - Connection drop test: terminate primary DB connection during multi-step transaction — verify lock/orphan-row behavior
  - Downstream failure test: take down least-trafficked downstream service — verify critical path isolation
  - Health check isolation test: partition so health checks pass but business traffic drops — verify real connectivity checks
Target domains: Infra | Backend
Blind spot: Application-layer logic bugs under load

## 15. Configuration Analyst — Environment and Drift Tester
Focus: Testing environment variable handling, configuration drift, and deployment artifact integrity
Test techniques:
  - Empty vs absent test: set required env var to `""` instead of absent — verify validation catches empty strings
  - Mid-lifecycle rotation: rotate secret during a long-running job — verify handling of changed credentials
  - Stale artifact test: serve deployment artifact from 2 commits ago — verify health check reports correct version
  - Feature flag test: apply prod-only feature flag never replicated to staging — verify parity
Target domains: DevOps | Infra | All
Blind spot: In-flight session behavior when config changes beneath active sessions

---

## CLUSTER VI — LOGIC TESTING (Business Logic, State Machines, Temporal)

## 16. Business Logic Analyst — Financial and Transactional Tester
Focus: Testing financial calculations, value transfers, and sequential validation logic
Test techniques:
  - Negative quantity test: `-1` units in cart, refund, or stock adjustment — verify totals can't go negative
  - Coupon stacking test: apply same coupon twice in parallel — verify deduplication
  - Threshold manipulation test: reach free-shipping threshold, remove items — verify conditional benefits update
  - Partial refund test: partial refund twice on same line item — verify validation against remaining balance
Target domains: Backend
Blind spot: Presentational bugs and network-layer issues

## 17. State Machine Analyst — Workflow Sequence Tester
Focus: Testing workflow sequencing, state transitions, and completion re-entry handling
Test techniques:
  - Step excision test: direct API call to final confirmation endpoint without completing intermediate steps
  - Backwards traversal test: complete step 5, re-submit step 3 with modified data — verify finalized output integrity
  - Double-completion test: submit terminal action twice rapidly — verify idempotent handling
  - Invalid transition test: PUT entity directly to `shipped` from `pending_payment` — verify transition validation
Target domains: Backend
Blind spot: Data at rest — tests motion of state, not storage

## 18. Temporal and Precision Analyst — Time and Locale Tester
Focus: Testing time-dependent logic, floating point arithmetic, and locale-sensitive handling
Test techniques:
  - Temporal boundary test: DST transition instants, Feb 29 in non-leap year, Unix timestamp `2147483647`, midnight UTC edge
  - Float accumulation test: repeated `0.1` additions on financial totals — verify rounding error handling
  - Multi-byte field test: single Japanese/Arabic character (3-4 UTF-8 bytes) in every `maxlength` or `VARCHAR(n)` field
  - Locale format test: European `1.234,56` in US fields; Turkish dotless-i `ı` uppercase — verify locale-safe comparisons
Target domains: All
Blind spot: Requires knowing deployment timezone and locale

---

## CLUSTER VII — DEEP SPECIALISTS

## 19. Cryptographic Analyst — Crypto Implementation Tester
Focus: Testing cryptographic implementation correctness, entropy quality, and timing safety
Test techniques:
  - Entropy audit: test RNG seeding during cold start / container initialization — verify unpredictable output
  - Timing analysis: measure all secret comparisons for constant-time behavior — detect microsecond latency deltas
  - Key lifecycle test: verify key material is zeroed after use, doesn't survive heap dumps or exception logs
  - Algorithm downgrade test: probe negotiated handshakes for `alg:none` acceptance, RSA/ECDSA confusion, legacy fallback
Target domains: Auth | API/Backend | Crypto
Blind spot: Application-layer logic bugs unrelated to cryptographic primitives

## 20. Regression Analyst — Behavioral Contract Tester
Focus: Testing whether behavioral contracts still hold after refactoring and changes
Test techniques:
  - Contract archaeology: extract implicit behavioral contracts from git history — verify code still honors them
  - Semantic diff test: find changes where diff looks safe but silently inverts behavior on null/zero/empty-string boundaries
  - Golden output test: replay production-captured request/response pairs against new build — flag any output divergence
  - Dependency shadow test: re-run historical test suite against each minor version of bumped deps — map behavioral drift
Target domains: API contracts | Serialization | DB query ordering | Error messages | All
Blind spot: Genuinely new features — only tests regressions in existing surface area
