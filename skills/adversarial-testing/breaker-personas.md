# Breaker Personas

## 1. The Chaos Monkey
Throws random, boundary, and malformed inputs at every entry point.
- Tests: null, undefined, empty string, max int, negative numbers, unicode, emoji, SQL in strings, HTML in text fields, arrays where scalars expected, deeply nested objects
- Goal: crash it, corrupt state, or trigger unhandled exceptions

## 2. The Pentester
Probes for security vulnerabilities with authorized testing mindset.
- Tests: injection (SQL, XSS, command), auth bypass, privilege escalation, IDOR, path traversal, exposed secrets in logs/errors, missing rate limiting, CSRF
- Goal: gain unauthorized access, leak data, or execute unintended operations

## 3. The Load Tester
Simulates concurrent access and resource pressure.
- Tests: concurrent writes to same resource, race conditions in read-modify-write, file handle exhaustion, memory pressure with large inputs, timeout behavior, connection pool exhaustion
- Goal: find deadlocks, data corruption under concurrency, or resource leaks

## 4. The Compatibility Checker
Tests across environments, versions, and configurations.
- Tests: different OS path separators, different Node/Python/runtime versions, missing optional dependencies, empty config files, read-only filesystem, different timezones, different locales
- Goal: find assumptions that only hold in the developer's environment

## 5. The Regression Hunter
Runs existing tests and checks unchanged functionality.
- Tests: full existing test suite, manual smoke test of unmodified features, verify no console warnings/errors introduced, check bundle size / dependency count didn't spike
- Goal: find things that USED to work but now don't
