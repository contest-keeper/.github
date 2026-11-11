# Contributing

Thanks for helping. These rules apply to every repository in the organization.

## Branches

Work on a short-lived branch, never on `main`:

- `feature/<name>` - new behaviour
- `fix/<name>` - bug fixes
- `chore/<name>` - tooling, dependencies, cleanup
- `docs/<name>` - documentation only
- `epic/<name>` - longer work that collects several PRs before going to `main`

Names are lowercase with dashes, e.g. `feature/esp-config-reader`.

## Commits and PR titles

We use [Conventional Commits](https://www.conventionalcommits.org) without scopes:

```
feat: add ESP config reader
fix: retry heartbeat after network drop
```

Types: `feat`, `fix`, `perf`, `refactor`, `docs`, `test`, `build`, `ci`, `chore`.
Lowercase, imperative ("add", not "added"), no period at the end.
Breaking changes use `feat!:` or `fix!:`.

PRs are squash-merged, so the **PR title** becomes the commit on `main` and is checked by CI.
Commits inside a branch can be anything.

## Pull requests

- One change per PR, kept small.
- CI must pass: formatting, lints (warnings are errors) and tests.
- Fill in the PR template and link the issue or plan step.

## Releases

Each repository is versioned with [SemVer](https://semver.org). Releases are cut by
release-please: it keeps a release PR open with the changelog built from `feat` and `fix`
commits, and merging that PR tags the version and runs the release pipeline.

## Code

Read the [code conventions](conventions/README.md). Before pushing:

- Rust: `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings`, `cargo test`, `cargo deny check`.
- TypeScript / Svelte: `pnpm format`, `pnpm lint`, `pnpm check`, `pnpm test`.
- Shell: `shellcheck` and `shfmt -i 2 -ci`.

Every repository has an `.editorconfig`; files use LF line endings.
