# Contributing

Thanks for helping. These rules apply to every repository in the organization.

## Branches

Create short-lived branches from an up-to-date `develop`, never commit directly
to `main` or `develop`:

- `feature/<name>` - new behaviour
- `fix/<name>` - bug fixes
- `chore/<name>` - tooling, dependencies, cleanup
- `docs/<name>` - documentation only
- `epic/<name>` - longer work that collects several PRs before going to `develop`

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

Feature PRs are squash-merged into `develop`, so the **PR title** becomes the commit
on `develop` and is checked by CI. Promotions from `develop` into `main` use merge
commits to preserve ancestry between the two long-lived branches.
Commits inside a branch can be anything.

## Pull requests

- One change per PR, kept small.
- CI must pass: formatting, lints (warnings are errors) and tests.
- Fill in the PR template and link the issue or plan step.

## Promoting develop to main

- Open feature and maintenance PRs against `develop`.
- Open a promotion PR from this repository's `develop` branch into `main`.
- Both branches require passing PR title, common CI, and branch policy checks,
  plus resolution of review conversations. Direct pushes, force pushes, and
  branch deletion are blocked, including for administrators in normal operation.
- Feature PRs use squash merges; promotions use merge commits, not squash or rebase.
- Keep both branches after merging. `main` remains the stable default branch.
- Review changes and test results before merging. No independent approval is
  mandatory while the organization has only one maintainer.
- A synchronization PR from `main` into `develop` is allowed when needed.
- Never include co-author trailers in commit or merge messages.

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
