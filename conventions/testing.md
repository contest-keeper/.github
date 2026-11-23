# Testing

## What to test

Test the key things, not everything. A small suite that covers what matters beats a large one that
buries the code. Every test must earn its place: if it would not catch a real mistake, don't write it.

- **Test:** logic where a mistake is costly or easy to make (signing, matching, parsing input from outside,
  validation rules, security checks), the main behaviour of each feature, and the failure cases that matter.
- **Don't test:** plain data types, getters, defaults, derived code (`serde`, `Debug`), simple wrappers,
  or anything the compiler already guarantees. One round-trip test per wire format is enough.
- One test per behaviour. Don't write many near-identical tests: use a table of cases inside one test.
- **Pure logic** (rules, parsing, matching, signing): unit tests next to the code.
- **Boundaries** (HTTP handlers, database, ICAP, file formats): a few integration tests in `tests/`, against
  a real PostgreSQL (container in CI), never a mock database.
- **Whole system** (stick boots, reports, Hub matches): the VM test in `ck-os`; run locally, not in CI,
  because it needs the contest image.
- Every bug fix comes with a test that fails without the fix.

## The keep rule

A test stays only if all three are true. Ask them about each test, in this order:

1. **Invariant.** It protects a rule the product must keep (a signature is checked, a login is rejected, a file
   is not overwritten). You can say the rule in one sentence.
2. **Plausible breaking change.** You can name a realistic edit that would break the rule, for example
   "someone drops the expiry check while refactoring". If you cannot, the test guards nothing.
3. **Only this test.** If that edit were made, no other test would fail. If another test already would,
   this one is a duplicate.

A test that fails any of the three is cut (or merged into the test that already covers it).

## Cut categories

Use these when removing a test, so a reviewer can check the reason at a glance.

| Id | Cut because the test... |
|---|---|
| C1 | tests a dependency or the standard library, not our code |
| C2 | tests a test helper, fixture or mock instead of the product |
| C3 | asserts a default or an optional field that nothing depends on |
| C4 | restates the type system or derived code (equality, `Debug`, a second `serde` round trip) |
| C5 | is a tautology: it compares a value with itself or with what it just built |
| C6 | duplicates a stronger test, in the same crate or another one |
| C7 | covers code or behaviour that was removed, replaced or is no longer wanted |
| C8 | cannot give a stable answer (timing, network, ordering) and is not worth making stable |

## Fix categories

A test that passes the keep rule but is built badly is fixed, not cut.

| Id | The test... | Fix |
|---|---|---|
| F1 | passes silently: it returns early when a database, tool or environment variable is missing | skip with a printed reason, and fail when `*_REQUIRED` is set (see below) |
| F2 | depends on the real clock, the network, the order of other tests or shared state | inject time, use a local fake server, give the test its own state |
| F3 | has a name that does not say the behaviour (`test_verify_2`) | rename it to the behaviour |
| F4 | asserts too little (`is_ok()`, a status code only) | assert the outcome: the error variant, the body, the stored row |
| F5 | is one of several near-identical tests | merge them into one table of cases |
| F6 | uses real secrets, cookies, names or unstripped traffic as fixtures | replace with generated or stripped data |

## Verdict lines

A test review reports one line per test, then the PR lists every removed test:

```
crates/ck-hub/src/ids.rs:108 tokens_compare_by_value CUT C4
crates/ck-hub/tests/enrol.rs:42 rejects_expired_claim KEEP
crates/ck-hub/tests/store.rs:17 stores_event FIX F1
```

Start from the code, not from the tests: list what could break unnoticed, map each test to that list, and
note what has no test. Cuts and fixes go in separate PRs, one per crate, and the PR body lists each removed test with its verdict line.

## Layout

- One integration test binary per crate: `tests/it/main.rs`, with the former files as modules (`mod enrol;`).
  Many small binaries link slowly and hide shared helpers.
- Shared fixtures and helpers live in `tests/it/support/`, not copied between files.

## Skipping and the `*_REQUIRED` rule

Tests that need an outside resource (a database, `qemu`, a signing tool) must never pass by doing nothing.

- Without the resource, the test prints why it was skipped and returns.
- When the matching `<NAME>_REQUIRED=1` variable is set, a missing resource is a **failure**. CI sets it
  (for example `CK_TEST_DATABASE_REQUIRED=1` next to `CK_TEST_DATABASE_URL`) and starts the resource as a service.
- Put the check in one helper per crate so every test skips or fails the same way.
- Slow or hardware tests use `#[ignore = "reason"]` and are run on purpose.

## Hostile input

Code that reads bytes from outside (JSON from a stick, a GPT or FAT image, the ICAP parser, HTTP bodies,
enrolment and heartbeat requests) gets a sweep with hostile input, not only hand-picked cases:

- Property tests (`proptest`) feed random and mutated input and assert one thing: it returns an error or a value,
  and never panics, hangs or allocates without a limit.
- Keep a small corpus of known bad inputs (truncated, oversized, wrong encoding, nested too deep) as ordinary tests.
- All `proptest` suites read their case count from `PROPTEST_CASES`, so CI and a developer machine set one budget.
- One test per security rule: for every item in a security review, a test that fails if the rule is removed.

## How

- Rust: `cargo test` (CI may use `cargo nextest`); `#[cfg(test)] mod tests` in the same file for unit tests.
  Use `insta` for snapshot tests of larger outputs and `proptest` for parsers and signing.
- Web: `vitest` for logic, Playwright for a few end-to-end flows of the dashboard.
- Test names say the behaviour: `rejects_event_with_bad_signature`, not `test_verify_2`.
- Tests don't depend on each other, on the clock (inject time) or on the network to outside services.
- Real traffic samples used as fixtures are stripped of passwords, cookies, session ids and names first.

## CI

- Formatting, lints and tests run on every PR and must pass.
- Slow or hardware tests are marked (`#[ignore]` with a reason) and run on purpose, not skipped silently.
- A test that is skipped in CI because its resource is missing is a bug in CI (see the `*_REQUIRED` rule).
