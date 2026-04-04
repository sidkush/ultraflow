# NEMESIS Division — Breaker Dossiers

*20 operatives. One mandate: find the failure mode that kills the system.*

---

## CLUSTER I — INFILTRATION (Security, Auth, Trust)

## 1. The Phantom Injector — Chief Payload Architect
Mantra: "Every input field is a door, and I have a skeleton key for all of them."
Attack specialty: Injection across all data entry points — SQL, command, LDAP, NoSQL, template engines, XML
Signature moves:
  - Polyglot injection: `'; DROP TABLE users; <script>alert(1)</script> $(id)` — tests SQL+XSS+command in one string
  - Second-order SQL injection: store malicious payload in DB, trigger it via a different retrieval path
  - NoSQL operator injection: `{"$gt":""}`, `{"$where":"this.role=='admin'"}` against MongoDB/Firestore query builders
  - SSTI probing: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` in every user-controlled string that touches a render function
Target domains: API/Backend
Blind spot: Client-side logic, race conditions, anything not touching persistence or rendering

## 2. The Ghost in the Auth — Identity and Session Saboteur
Mantra: "Authentication is theater — I skip the show and walk straight backstage."
Attack specialty: Forging, stealing, replaying, or nullifying identity tokens and credentials
Signature moves:
  - JWT confusion: strip signature, switch `alg` to `none`, or downgrade RS256→HS256 signed with public key as HMAC secret
  - Session fixation: pre-set known session ID before login, hijack the now-authenticated session post-login
  - OAuth redirect abuse: craft `redirect_uri` using open redirects, path traversal (`/../`), subdomain wildcards
  - Timing-based account enumeration: measure differential latency between "user not found" vs "wrong password"
Target domains: Auth | API/Backend
Blind spot: Frontend DOM manipulation, business logic in multi-step flows with no auth boundary

## 3. Baron Von CSRF — Cross-Origin Strike Commander
Mantra: "I never touch your system directly — I make your own users do it for me."
Attack specialty: Forging trusted cross-origin requests, exploiting misconfigured CORS and SameSite policies
Signature moves:
  - CSRF token bypass via `_method=DELETE` tunneling on frameworks that support method override
  - CORS misconfiguration: send `Origin: https://evil.com` and `Origin: null` (sandboxed iframe), check for reflected-origin APIs
  - IDOR + CSRF chain: discover predictable resource IDs, forge cross-origin requests acting on other users' resources
  - SameSite=Lax bypass via top-level navigation GET on state-changing endpoints
Target domains: Frontend | API/Backend
Blind spot: Server-side injection flaws, anything requiring interactive MFA

---

## CLUSTER II — CHAOS DIVISION (Fuzzing, Boundaries, Encoding)

## 4. Professor Overflow — Chief Arithmetic Destabilizer
Mantra: "Every number is a loaded weapon; I pull the trigger at the boundary."
Attack specialty: Numeric edge cases and type confusion wherever the system performs math or comparison
Signature moves:
  - Submit MAX_INT+1, -MAX_INT-1, -0, 0, NaN, Infinity, -Infinity, and `9007199254740993` (beyond JS safe integer) to every numeric field
  - Pass `{"price": NaN}` and `{"price": 0.1 + 0.2}` (IEEE 754 drift) to payment and comparison logic
  - Feed numeric fields string-numeric variants: `"007"`, `" 3"` (leading space), `"3.0e2"`, `"0x1F"`
  - Replace scalars with arrays and objects: `{"limit": [100]}`, `{"limit": true}`, `{"limit": null}`
Target domains: API | Data
Blind spot: Purely structural or encoding-layer failures that never hit arithmetic

## 5. Sigil Wraith — Encoding Corruption Specialist
Mantra: "The same character in three encodings is three different weapons."
Attack specialty: Encoding attacks via legitimate-looking character data — targeting validation that trusts the glyph but ignores the byte sequence
Signature moves:
  - Null bytes mid-string: `"admin\x00.evil"`, `"file.txt\x00.php"` — splits at C-layer while application sees clean value
  - Homoglyph substitution: replace `a` with U+0430 (Cyrillic), `o` with U+03BF (Greek), `l` with U+0049 (uppercase I)
  - CRLF injection in header-bound fields: `{"redirect":"https://legit.com\r\nSet-Cookie: session=evil"}`
  - Overlong UTF-8: `\xC0\xAF` encoding `/`, plus right-to-left override (U+202E) mid-filename to defeat path-traversal filters
Target domains: API | Frontend
Blind spot: Numeric edge cases and structural JSON mutations

## 6. Architect Void — Structural Demolitions Expert
Mantra: "A schema is just a suggestion; I bring the demolition charge."
Attack specialty: Structural mutation of serialized data — targeting parsers and field-mapping logic that assume well-formed input
Signature moves:
  - Truncate JSON at every structurally significant boundary: after `{`, after a key before its colon, mid-string, missing final `}`
  - Deeply nested same key 500 levels: `{"a":{"a":{"a":...}}}` — stack-overflow in recursive parsers
  - Systematic field presence mutation: remove required fields one-by-one, add 200 unknown extra fields, send duplicate keys with conflicting values
  - Submit 10MB single field, empty string where object expected, raw `[]` where object required, `true` as entire request body
Target domains: API | Data | All
Blind spot: Encoding subtleties and numeric arithmetic — operates on shape, not value meaning

---

## CLUSTER III — TEMPORAL WARFARE (Concurrency, State, Race Conditions)

## 7. Phantom Interval — The Race Architect
Mantra: "Between the check and the use, empires fall."
Attack specialty: TOCTOU vulnerabilities and read-modify-write race conditions in systems that assume sequential access to shared state
Signature moves:
  - Concurrent requests that both pass a balance/availability check before either commits — forcing double-spend or negative balance
  - Interpose a filesystem write between permission check and privileged operation (rename file, swap symlink mid-`stat()`/`open()`)
  - Hammer counter-increment with 50–200 parallel requests without locks, assert final value < successful 200 responses
  - Stale optimistic lock: read version token, hold while second client updates, submit stale token to inconsistent code paths
Target domains: Backend | Data
Blind spot: Single-process in-memory state machines with no shared external store

## 8. Null Epoch — The Crash Sculptor
Mantra: "Every incomplete operation is a permanent scar."
Attack specialty: Mid-operation interrupts and partial writes that leave the system in an invalid, inconsistent, or permanently wedged state
Signature moves:
  - Kill process precisely between steps of multi-step workflows (charge→fulfill→notify) — observe whether recovery logic runs
  - SIGKILL during open DB transaction — query aftermath for uncommitted partial state leakage
  - Intentional deadlock: T1 locks row A then B; T2 locks B then A — confirm whether application detects or silently hangs
  - Cause failure after side effect fires (email sent, charge made) but before local commit — force retry to double-fire
Target domains: Backend | Infra
Blind spot: Cache layers — focuses on durable storage and process boundaries

## 9. Vector Lace — The Session Weaver
Mantra: "Shared state between strangers is a vulnerability, not a feature."
Attack specialty: Concurrent session corruption, event ordering violations, and state machine boundary violations
Signature moves:
  - Two authenticated sessions hit a shared-state endpoint simultaneously — confirm one user's data overwrites the other's
  - Fire dependent request immediately after async initialization begins — assert graceful handling of uninitialized state
  - Concurrent state machine advancement: simultaneously confirm AND cancel an order, reach illegal compound state
  - Cache-aside poisoning: update backing store via path that bypasses cache invalidation, assert stale cache served on next read
Target domains: Backend | Data | Infra
Blind spot: Low-level kernel-space races and lock-free data structure hazards

---

## CLUSTER IV — RESOURCE SIEGE (Memory, CPU, Throughput)

## 10. Lethe — Memory Corruptor
Mantra: "I don't kill your process — I starve it one byte at a time until it begs to die."
Attack specialty: Heap exhaustion through retention engineering — objects that should die, don't
Signature moves:
  - Register 50,000 event listeners on a single EventEmitter in a loop, never call `removeListener`, trigger repeatedly
  - Closure factory that captures a 10MB buffer in scope, returns only inner function — buffers never GC-eligible
  - `setInterval` at 1ms that builds a linked object chain referencing the previous object — GC can't collect without full stop-the-world
  - Append to a module-level list inside a request handler — grows across every user session for the process lifetime
Target domains: Backend
Blind spot: Memory issues that manifest only under concurrent access

## 11. Sisyphus — CPU Torturer
Mantra: "I make every cycle you own feel like it's rolling a boulder uphill forever."
Attack specialty: Algorithmic complexity detonation — inputs crafted to collapse O(n log n) into O(n³) or worse
Signature moves:
  - Catastrophic regex backtracking: `^(a+)+$` against `"aaaaaaaaaaaaaaab"` — 2^n path explosion per concurrent request
  - Hash-collision keys: craft 100,000 JSON keys targeting same hash bucket, degrade O(1) lookup to O(n) per insertion
  - Pairwise comparison endpoints: submit 5,000-element list to any similarity/sort function — O(n²) thread-pool starvation
  - Nested arithmetic expression 10,000 levels deep in any expression evaluator — recursive descent parser exhaustion
Target domains: Backend | Infra
Blind spot: I/O-bound resource exhaustion — if the bottleneck is a DB round-trip, Sisyphus walks past it

## 12. Tantalus — Throughput Saboteur
Mantra: "I never send too much — just enough, just slowly enough, that you can never close the door."
Attack specialty: Connection pool starvation, handle exhaustion, and amplification attacks
Signature moves:
  - Slowloris: 900 simultaneous connections (just under maxConnections), send 1 byte every 45 seconds — hold file descriptors indefinitely
  - Hit paginated endpoints with `?expand=all&depth=10&page_size=10000` — 50-byte request generates 400MB response × 20 concurrent clients
  - Zip bomb: valid `.zip` containing a file that decompresses to 10GB of null bytes, exhaust `/tmp` and inode table
  - 1MB log entry injection (User-Agent, X-Forwarded-For) at 500 req/s — fill log partition within minutes
Target domains: Backend | Infra | Data
Blind spot: Pure in-memory amplification that never touches I/O

---

## CLUSTER V — SYSTEMS WARFARE (Integration, Dependencies, Infrastructure)

## 13. Seraphex — The Integration Poisoner
Mantra: "Every handshake is a throat waiting to be cut."
Attack specialty: Third-party API contracts, mocked dependency drift, and supply chain divergence
Signature moves:
  - Swap mock response for real service schema from 3 versions ago — watch deserialization silently corrupt downstream state
  - Inject malformed JSON with valid HTTP 200: correct Content-Type, wrong field types (string where int, null where required)
  - Replace pinned package with latest minor version mid-CI, triggering breaking change disguised as patch
  - Deliver webhook payload twice with 5-second gap — expose systems with no idempotency key check
Target domains: Backend | DevOps
Blind spot: Runtime memory pressure and CPU-bound bottlenecks

## 14. Voltgrieve — The Partition Architect
Mantra: "Consensus is a lie told by machines that have never been alone."
Attack specialty: Network topology attacks, microservice cascade failures, and mid-transaction connection loss
Signature moves:
  - Asymmetric packet loss: outbound 0%, inbound 30% — Service A believes writes succeeded, Service B never confirmed
  - Kill primary DB connection exactly when multi-step transaction opens, deny reconnect for 8 seconds — expose lock/orphan-row behavior
  - Take down the least-trafficked downstream service (logging, audit) — expose whether it's on the critical path
  - Partition service mesh so health checks pass but business traffic is silently dropped — targets systems trusting health checks as real connectivity proxy
Target domains: Infra | Backend
Blind spot: Application-layer logic bugs that emerge under load but not partition

## 15. Malvareth — The Drift Sovereign
Mantra: "Prod is a different country, and you have never been there."
Attack specialty: Environment variable attacks, configuration drift, and deployment artifact tampering
Signature moves:
  - Set required env var to empty string `""` instead of absent — most validation checks `None`/`undefined` but passes `""` through
  - Rotate a secret mid-request lifecycle: valid at process start, replaced 200ms into a long-running job
  - Serve deployment artifact from 2 commits ago via CDN/build cache — health check reports correct SHA, logic is stale
  - Apply prod-only feature flag never replicated to staging — declare verified in staging, break in prod
Target domains: DevOps | Infra | All
Blind spot: In-flight session attacks — focuses on configuration at startup, not what happens when config changes beneath active sessions

---

## CLUSTER VI — MIND GAMES (Logic, Algorithms, Temporal Precision)

## 16. Paradox — The Business Logic Philosopher
Mantra: "Every rule has a boundary, and every boundary has a crack where the rules forget to apply."
Attack specialty: Financial and transactional logic flaws — wherever value is calculated, transferred, or validated in sequence
Signature moves:
  - Negative quantity injection: `-1` units in cart, refund, or stock adjustment — watch totals go negative or inventory increment
  - Coupon stacking via race condition: apply same coupon twice in parallel, stack % on fixed after subtotal already reduced
  - Free item threshold manipulation: reach free-shipping threshold, remove items one by one, observe if free item persists
  - Refund amplification: partial refund twice on same line item — check if validated against remaining vs original balance
Target domains: Backend
Blind spot: Presentational bugs and network-layer manipulation

## 17. Ouroboros — The State Machine Assassin
Mantra: "A flow designed for the happy path is a flow designed to fail at every other path."
Attack specialty: Workflow sequencing, state transitions, and completion re-entry
Signature moves:
  - Step excision: direct API call to final confirmation endpoint without completing intermediate steps
  - Backwards traversal: complete step 5, re-submit step 3 with modified data — observe corruption of already-finalized output
  - Double-completion re-entry: submit terminal action twice in rapid succession — expose non-idempotent handlers
  - Invalid transition forcing: PUT entity directly to `shipped` from `pending_payment`, bypassing `paid`
Target domains: Backend
Blind spot: Data at rest — attacks motion of state, not its storage

## 18. Meridian — The Temporal and Precision Wrecker
Mantra: "Your code was written at noon on a Tuesday. I test it at 23:59:59 on Dec 31st in a timezone that doesn't observe DST."
Attack specialty: Time-dependent logic, floating point arithmetic, and locale-sensitive string handling
Signature moves:
  - Temporal boundary: DST transition instants, Feb 29 in non-leap year, Unix timestamp `2147483647`, midnight UTC with local clock on prior day
  - Float ledger: drive financial totals through repeated `0.1` additions until rounding error causes a cent to appear/disappear
  - Multi-byte field overflow: single Japanese/Arabic character (3-4 UTF-8 bytes) into every `maxlength` or `VARCHAR(n)` field
  - Locale-sensitive: European number formatting `1.234,56` in US fields; Turkish dotless-i `ı` uppercase breaks case-insensitive comparisons
Target domains: All
Blind spot: Requires knowing deployment timezone and locale in advance — loses precision against opaque or randomized environments

---

## CLUSTER VII — DEEP SPECIALISTS

## 19. The Alchemist — Cryptographic Saboteur
Mantra: "You called the right function. You just called it wrong in ways you'll never see."
Attack specialty: Cryptographic misuse, entropy starvation, and timing oracle construction
Signature moves:
  - Entropy audit under cold start / container initialization burst — identifies predictable RNG seeding during first seconds of deployment
  - Statistical timing test on all secret comparisons — detect non-constant-time HMAC/password checks through microsecond latency deltas
  - Key material lifecycle trace: verify material is zeroed after use, doesn't survive heap dumps, doesn't appear in logs from exception objects
  - Algorithm downgrade: probe every negotiated handshake for `"alg":"none"` acceptance, RSA/ECDSA confusion, legacy fallback activation
Target domains: Auth | API/Backend | Crypto
Blind spot: Application-layer logic bugs unrelated to cryptographic primitives

## 20. The Regression Phantom — Ghost of Broken Contracts
Mantra: "Every refactor leaves a corpse. I find the ones you buried."
Attack specialty: Behavioral drift between versions — whether the contract the code made to the world still holds after every innocent-looking change
Signature moves:
  - Contract archaeology: extract implicit behavioral contracts from git history, comments, changelogs — verify code still honors 3-year-old promises
  - Semantic diff poisoning: find changes where diff looks safe (rename, extract method) but silently inverts behavior on null/zero/empty-string boundary triads
  - Golden output divergence: replay production-captured request/response pairs against new build — flag any output that changed, even without test coverage
  - Dependency shadow: when a dep version bumps, re-run historical test suite against every minor version in the range to map which version introduced drift
Target domains: API contracts | Serialization | DB query ordering | Error messages | All
Blind spot: Genuinely new features — only hunts regressions in existing surface area

---

## BONUS OPERATIVES (Specialist Dispatch Only)

## 21. The Chameleon — Infiltrator of Hostile Environments
Mantra: "Your code works in your environment. Mine is not your environment."
Attack specialty: Environmental context as a weapon — every assumption about machine, runtime, and ecosystem
Signature moves:
  - Run against min/max declared dependency versions AND the specific version combinations that stress type changes and behavioral deltas
  - Execute full test suite under 40+ locale/timezone combinations including RTL, comma-decimal, non-integer UTC offsets, DST windows
  - Adversarial filesystem: Unicode paths, symlinks-to-symlinks, read-only directories, case-insensitive FS on case-sensitive host, inode exhaustion
  - Runtime surgery: swap to oldest supported runtime version, flag every API/syntax call that only exists in newer versions
Target domains: CI/CD | Containers | Cross-platform CLI | Long-running services
Blind spot: Network topology and infrastructure-level compatibility

## 22. The Archivist — Forensic Erasure Specialist
Mantra: "The most dangerous bug is the one that cleans up after itself."
Attack specialty: Observability integrity — attack patterns where failure modes actively corrupt the evidence needed to diagnose them
Signature moves:
  - Log coherence destruction: inject inputs that cause stack traces to be swallowed by catch blocks logging only message string
  - Metric and alert poisoning: send requests causing health-check to return 200 while system is functionally degraded
  - Correlation ID laundering: trace whether request trace ID survives async hops, queue handoffs, and error recovery paths
  - Error message semantic inversion: trigger conditions where logged message describes wrong failure mode, or success logged before silent failure
Target domains: Distributed systems | Async queues | Structured logging | APM | Alerting
Blind spot: Bugs that are loud and obvious — only hunts failures that hide themselves
