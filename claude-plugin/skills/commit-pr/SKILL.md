---
name: commit-pr
description: Commit the current work and open a pull request the Contest Keeper way (branch names, Conventional Commit titles, short plain text). Use for "/ck:commit-pr", "commit this", "open a PR".
---

# Commit and PR

The conventions are in `CONTRIBUTING.md` of `contest-keeper/.github`. In short:

## 1. Branch

Never commit to `main`. If on `main`, create a branch first:
`feature/`, `fix/`, `chore/`, `docs/` or `epic/` + lowercase-dashed name.

## 2. Checks

Run the repository's checks before committing and fix what fails
(Rust: `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings`, `cargo test`;
web: `pnpm lint`, `pnpm check`, `pnpm test`).

## 3. Commit

- Stage only files that belong to this change. Never stage secrets, `.env` files, disk images or build output.
- Message: `type: subject` - type is one of `feat fix perf refactor docs test build ci chore`,
  no scope, lowercase imperative subject, no period, under 72 characters.
  Breaking change: `feat!:` / `fix!:`.
- Commit as the user. Do **not** add `Co-Authored-By` trailers, "Generated with" lines or any other
  tool attribution - this project's texts must read as written by its maintainers.

## 4. Pull request

- Push the branch, then `gh pr create`.
- The PR title follows the same `type: subject` format - it becomes the squash commit on `main`
  and CI checks it.
- Body: the repository's PR template (What / Why / Closes / How it was tested). Plain, short
  sentences; no emoji, no marketing, no headings beyond the template's.
- Link the plan step (e.g. "Plan step A.3") or issue under Why.
- Show the user the title and body before creating the PR.

## 5. After

Tell the user the PR link. Do not merge it; the user merges after CI passes.
