# Rust

## Toolchain

- Edition 2024. The version is pinned in `rust-toolchain.toml` (see `templates/rust/`); bump it in its own `chore:` PR.
- `cargo fmt`, `cargo clippy --all-targets -- -D warnings` and `cargo deny check` must pass.
- Lints are set once per workspace in `Cargo.toml` (`[workspace.lints]`, see `templates/rust/Cargo.toml`)
  and every crate has `[lints] workspace = true`.

## Layout

- One Cargo workspace per repository. Crates live in `crates/<name>`, binaries in `crates/<name>` with a `main.rs`.
- Crate names: `ck-<thing>` (`ck-protocol`, `ck-agent`). Module and file names: `snake_case`.
- Split by responsibility, not by type: `login.rs`, `uplink.rs` rather than `models.rs`, `utils.rs`, `helpers.rs`.
- Keep `main.rs` short: parse config, build the parts, run. Logic goes in the library part of the crate.
- Pure logic (rules, parsing, matching, signing) has no I/O and no async, so it is easy to test.

## Types

- Make wrong states impossible: newtypes for ids (`StickId(String)`), enums instead of strings or bools.
- Parse, don't validate: turn raw input into checked types once at the boundary
  (`StickId::parse(&str) -> Result<StickId, _>`), then trust the type inside.
- Public structs that cross the wire derive `Serialize`/`Deserialize` and live in `ck-protocol`.
- Prefer `&str`/slices in arguments, owned types in structs.

## Errors

- Libraries: one `Error` enum per crate (or module) with `thiserror`. Variants describe what went wrong, with context.
- Binaries: `anyhow` (or `color-eyre`) at the top level, with `.context("reading ESP config")`.
- No `unwrap()`/`expect()` outside tests, except for true invariants, and then `expect("why this can't fail")`.
- No `panic!` for expected failures (bad input, network down, missing file).

## Async and concurrency

- `tokio` for async. Don't block inside async code; use `spawn_blocking` for heavy or blocking work.
- Every network call has a timeout. Retries use backoff with a limit.
- Long-running tasks take a cancellation signal (`CancellationToken`) and shut down cleanly on SIGTERM.
- Prefer channels and owned state over shared `Mutex`es.

## Logging

- `tracing` everywhere, never `println!` in services. CLI output for humans may use `println!`.
- Levels: `error` = needs attention, `warn` = unexpected but handled, `info` = important state changes,
  `debug`/`trace` = details for developers.
- Structured fields, not formatted strings: `info!(stick = %id, "heartbeat sent")`.
- Never log secrets or personal data (see [security.md](security.md)).

## Style

- Comment sparingly, only at key points. Clear names and small functions come first; a comment is for what the code can't say:
  a hidden constraint, a surprising choice, a workaround, a subtle invariant. It explains *why*, never *what*.
- No comments that repeat the code, label sections or narrate steps.
- `///` on a public item only when the name and signature don't tell the whole story: units, edge cases,
  invariants, errors a caller must handle. Don't restate the name ("Returns the id"). Add a short example where it helps.
  Each library crate keeps a one-line `//!` that says what it is for.
- `unsafe` only with a `// SAFETY:` comment, and only if there is no safe way.
- Features and `cfg` flags are named in kebab-case and documented in the crate README.
