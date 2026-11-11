# Code conventions

These apply to every Contest Keeper repository. Git, PR and release rules are in
[CONTRIBUTING.md](../CONTRIBUTING.md).

| Page | Covers |
|---|---|
| [rust.md](rust.md) | Rust code: layout, errors, async, logging, lints |
| [web.md](web.md) | TypeScript and Svelte: dashboard, extension, Forge window |
| [api.md](api.md) | HTTP API, events between stick and hub, compatibility |
| [database.md](database.md) | PostgreSQL schema and migrations |
| [system.md](system.md) | Shell scripts, systemd units and the image patch |
| [testing.md](testing.md) | What to test and how |
| [security.md](security.md) | Secrets, personal data, logging, dependencies |
| [docs.md](docs.md) | READMEs, comments, decision records |

## The short version

1. **Keep logic pure, push I/O to the edges.** Rules like "is this a login?" or "does this
   submission match?" are plain functions with tests. Network, disk, clocks and processes are
   thin layers around them.
2. **Small things.** Small functions, small files, small PRs. A file over ~400 lines is a hint to split it.
3. **Names say what, not how.** `match_submission`, not `process_data`. No abbreviations except
   well-known ones (`id`, `url`, `http`, `ck`).
4. **Fail loudly and early.** Validate input at the boundary, return clear errors, never swallow one.
5. **No secrets, no personal data in code, logs or Git.** See [security.md](security.md).
6. **Test the key things**, at the level that matches them, not every line. See [testing.md](testing.md).
7. **English everywhere:** code, comments, docs, commit messages, UI strings (translations come later).
8. **Formatters decide formatting.** Don't argue style a tool can fix; run `cargo fmt` / `pnpm format`.
9. **Fewer dependencies.** Add a crate or package only when it saves real work and is maintained.
   Mention why in the PR.
10. **Time is UTC.** Store and send timestamps as UTC (RFC 3339); convert only when showing them.
