# Four parallel sessions (2026-11-23)

All open work is split into four sessions. Each owns whole repos, so no two sessions edit the same files. Inside a
session the steps run one by one, in order, and each step is implemented, opened as a PR and merged before the next starts
(rule 11). Start one Claude Code session per file with:

> Read `D:\contest-keeper\sessions\parallel\README.md` and `D:\contest-keeper\sessions\parallel\L<n>-<name>.md`, then do the first unfinished step.

| Session | File | Owns (only it edits these) |
|---|---|---|
| L1 | `L1-ck-os.md` | **ck-os**: image, tools, docs, agent crates, ICPC removal, signed-settings gate, security hardening, profiles |
| L2 | `L2-hub-and-infra.md` | **ck-hub** and **ck-infra** (and the Azure side of any check) |
| L3 | `L3-forge-and-protocol.md` | **ck-forge** and **ck-protocol** (with its crates, e.g. crypto and image) |
| L4 | `L4-process-and-housekeeping.md` | **`.github` repo**, **ck-website**, the workspace folder (`sessions/`, plans), the flows artifact, and, last, the cross-repo lints and `AGENTS.md` files |

Rule that keeps this safe: **a session never edits a repo another session owns.** L4 adds `AGENTS.md`,
`.coderabbit.yaml` and `CLAUDE.md` to the other repos only as its final step, after the owner's open PRs are merged,
one small PR per repo from fresh `origin/develop`. A session that needs a change in another session's repo writes one
line (from, what, why) under "Requests" at the bottom of the owner's file.

## Rules for every session (from the workspace and repo CLAUDE.md files)

1. Reply in English.
2. Ask which GitHub account to use (`sheila-dot` or `robbertpopa`) before any commit, push or `gh` command. Then
   `gh auth switch --user <account>` and set repo-LOCAL `user.name` / `user.email`:
   sheila-dot `270025227+sheila-dot@users.noreply.github.com`, robbertpopa `95902235+robbertpopa@users.noreply.github.com`.
3. Work in your own worktree, never the main checkout (it sits on an old feature branch). Fetch first:
   `git -C D:\contest-keeper\<repo> worktree add D:\ck-wt\L<n>-<repo> -b <type>/<name> origin/develop`.
4. Branch from `develop`, squash PR into `develop`, title `type: lowercase imperative subject`.
   **No `Co-Authored-By`, "Generated with" or other tool attribution lines** in ck-* commits or PRs
   (the repo CLAUDE.md overrides the harness default).
5. Before each commit: `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings`, `cargo test`, shellcheck for
   scripts (Docker). ck-hub tests need `CK_TEST_DATABASE_URL` pointing at a local PostgreSQL. ck-forge `ui/`: `pnpm lint check test`.
6. Never store, log or send passwords, cookies, tokens or the Wi-Fi password. Never touch `C:\rcpc\`.
7. Ask before deleting, force pushing, or anything that leaves the machine. Deploys go through the repo pipelines;
   never `terraform apply` or any `az` write from a session.
8. Decisions live only in `sessions/DECISIONS.md`; never restate or change one.
9. Moves and behaviour changes go in separate PRs; small PRs; run `/code-review high` on a PR and fix its findings before merging.
10. Do not start a step marked "waits for": it needs another session's PR merged first. Do the next step instead, and come back.
11. **Implement everything in your file, then finish each PR yourself.** Each checklist item ends with: push the branch, open the
    PR into `develop`, wait for CI to be green (fix failures, re-run flaky jobs once), squash-merge it (`gh pr merge --squash`,
    never `--admin`, never bypass a required check or review rule), delete only your own merged branch and worktree, and tick the
    box. If a required review or check cannot pass without a human (missing approval, a secret, a failing check you cannot fix),
    stop on that PR, say exactly what is needed, and continue with the next item that does not depend on it.
    Merging is allowed only for PRs you opened in this session; merge nothing else.
12. Merging into `develop` is the end of your authority: never promote `develop` to `main`, never tag or release, never run a deploy.
    (Pipelines run by themselves after a merge; watch the result and report it.)
13. At the end of each step: PR links, what ran and what did not. Tick the item in the session's own checklist and in
    `IMPLEMENTATION-PLAN.md` only for what is merged or verified.
14. Review findings (R4.1) go to `sessions/review-2026-11-<dd>-<repo>.md`, one file per repo, written by the owner.

## Needs the user (no session may do these)

- Organizer login on the dev Hub: G4 acceptance, real Azure recording and capacity checks (Hub and infra lane lists them).
- Real laptop boots and the Kingston USB write (hardware).
- Licence approval #31 and the Canonical question; cloud-cost approval C7.
- `RELEASE_PLEASE_TOKEN` secret and the CodeRabbit GitHub app install.
- Approving any deletion of branches, worktrees or folders.

## Plan source

`sessions/plan-review-and-craft.md` (R-ids), `IMPLEMENTATION-PLAN.md` ("State check against the code (2026-11-23)"),
decisions in `sessions/DECISIONS.md`.
