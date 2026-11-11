# Testing

## What to test

- **Pure logic** (rules, parsing, matching, signing): unit tests next to the code. Cover the edge cases
  and the failure cases, not only the happy path.
- **Boundaries** (HTTP handlers, database, ICAP, file formats): integration tests in `tests/`, against
  a real PostgreSQL (container in CI), never a mock database.
- **Whole system** (stick boots, reports, Hub matches): the VM test in `ck-os`; run locally, not in CI,
  because it needs the contest image.
- Every bug fix comes with a test that fails without the fix.

## How

- Rust: `cargo test`; `#[cfg(test)] mod tests` in the same file for unit tests. Use `insta` for
  snapshot tests of larger outputs and `proptest` for parsers and signing.
- Web: `vitest` for logic, Playwright for a few end-to-end flows of the dashboard.
- Test names say the behaviour: `rejects_event_with_bad_signature`, not `test_verify_2`.
- Tests don't depend on each other, on the clock (inject time) or on the network to outside services.
- Real traffic samples used as fixtures are stripped of passwords, cookies, session ids and names first.

## CI

- Formatting, lints and tests run on every PR and must pass.
- Slow or hardware tests are marked (`#[ignore]` with a reason) and run on purpose, not skipped silently.
