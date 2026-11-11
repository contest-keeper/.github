---
name: next-step
description: Pick up the next step of the Contest Keeper plan for this repository, implement it on its own branch, and tick it off. Use for "/ck:next-step", "/ck:next-step A.3", "next step", "continue the plan".
---

# Next plan step

The plan lives in the `ck-website` repository under `plan/`:

- `plan/README.md` - every step with a status checkbox, and which repository each phase's code lives in
- `plan/context.md` - shared facts, names, decisions and gotchas; read it every time
- `plan/<phase>/<step>.md` - one file per step: Why, Context, Small steps, Done when, Out of scope

Look for it in a sibling folder (`../ck-website/plan`). If it is not there, read it from GitHub
with `gh api repos/contest-keeper/ck-website/contents/plan/...` or ask the user where it is.

## Steps

1. If the user named a step (e.g. `A.3`), use it. Otherwise take the first unchecked step in
   `plan/README.md` whose code lives in the current repository and whose "Depends on" steps are done.
   Tell the user which step you picked and wait for a yes.
2. Read `context.md`, the step file, and the code conventions (`conventions/` in `contest-keeper/.github`) for the languages you touch. Do only what the step's "Small steps" describe; anything
   under "Out of scope" stays out.
3. Branch from an up-to-date `develop`: `feature/<step-id>-<short-name>` (e.g. `feature/a3-image-tool`),
   or `fix/`, `chore/`, `docs/` when that fits better.
4. Work through the small steps in order. After each one, build and run the checks
   (Rust: `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings`, `cargo test`;
   web: `pnpm lint`, `pnpm check`, `pnpm test`).
5. Check every item under "Done when". If one cannot be checked here (real hardware, a contest
   account, the user's password), say so instead of marking it done.
6. Hand over to `/ck:commit-pr` for the commit and PR.
7. In `ck-website`, tick the step in `plan/README.md` and write down anything new you learned in
   `context.md` (gotchas, decisions). That is its own small `docs:` PR.

## Rules

- Never type, store or log passwords, cookies, session ids or tokens. The user types credentials themselves.
- Ask before anything destructive or outward-facing (deleting, force pushing, transferring repos).
- If the step file is wrong or out of date, fix the step file in the same `ck-website` PR.
