# L4: process, conventions, housekeeping, artifact

Read `README.md` in this folder first (rules, including rule 11: implement each item, open the PR, wait for green CI and squash-merge it yourself). You own the **`.github` repo** (`D:\contest-keeper\.github`: check it is a git repo and
its default branch), **ck-website**, the workspace folder (`sessions/`, plan files) and the flows artifact. You touch the
other repos only in step 6 (new files only). Plan source: `sessions/plan-review-and-craft.md`. User decisions (2026-11-20):
`AGENTS.md` allowed and `CLAUDE.md` = `@AGENTS.md`; CodeRabbit is free for our public repos; do not copy Craft's GPL
clean-room, i18n or egui accessibility rules (decision 14 ships GPL programs).

## Checklist (in order; steps 1 and 2 first, because L1 to L3 wait for step 2)

### Step 1: housekeeping audit (read-only, deletes nothing)

Decision 12: merged branches and worktrees are pruned; unmerged ones and other lanes' worktrees are never deleted without asking.
Needs the account question (`gh pr list` is read-only but still `gh`).
- [x] Worktrees: `D:\ck-wt\*` (20 folders) and each repo's `git worktree list`: branch, last commit, dirty/clean, merged into
  `origin/develop` or not (`git merge-base --is-ancestor` and `gh pr list --state merged --head <branch>`, since squash merges hide
  ancestry), unpushed work. Dirty or unpushed means "keep, ask". Other sessions may use some: `git status` only, never write.
- [x] Local branches in the six repos, same classification; include the plan's candidates (ck-forge `feature/windows-updater`,
  `feature/auto-update`, `fix/show-checking-while-writing`; ck-hub `fix/review-fixes`, `feature/register-sticks`,
  `feature/organizer-accounts`; ck-os `feature/welcome-text`, `feature/image-patch-agent`).
- [x] Stray folders in `D:\contest-keeper`: `ck-hub-setup` (only a `.git`, a duplicate of ck-infra), `ck-os;C` (mis-typed), `_archive/ck-hub-setup`,
  `pipeline-scratch`, `review/`: what each is, size, whether anything is not elsewhere.
- [x] Disk: free space on `C:` and `D:`; sizes of `D:\ck-wt`, `D:\ck-images`, `target/`, `target-dev/`.
- [x] Open PRs in all 7 repos with CI state; green ones ready to squash-merge (do not merge).
- [x] Report to `sessions/housekeeping-2026-11-23.md` with a "Proposed deletions" list of exact commands (not run) for the user to approve; one line in the master plan pointing at it.
- [ ] (light re-run done after steps 2 to 5, see the addendum in the report) Re-run this audit at the end of each later step, and keep `IMPLEMENTATION-PLAN.md` and the lane files' ticks and "Status / next" lines true (this is the only session that edits them; others report under their own file).

### Step 2: `.github` repo conventions (L1 to L3 wait for the testing standard)

- [x] PR 1 (R3.1) (merged: .github#27): merge a testing standard into `conventions/testing.md`: the keep rule (invariant, plausible breaking change, only this test), cut categories C1-C8 and fix categories F1-F6 (define them, short), verdict lines (`file:line name CUT C3`), one integration binary per crate, the `*_REQUIRED` skip rule, hostile-input sweeps. Own words.
- [x] PR 2 (merged: .github#28) (R5.1 + R4.4): `conventions/agents.md` shared house rules (accountable person; check, then claim; verify, don't guess; narrow tested fixes; review the diff; report only what ran; no pushes to main; guards say why; the test rules). Add the comment rule to `conventions/rust.md`. Link from `conventions/README.md`.
- [x] PR 3 (merged: .github#29) (R5.2 template only): `templates/rust/` workspace lints and profiles (`unsafe_code = "forbid"`, `[profile.dev] debug = "line-tables-only"`, `[profile.dev.package."*"] opt-level = 2, debug = false`, nextest in CI). Do not apply to repos yet.
- [x] PR 4 (merged: .github#30): `house-rules` reusable workflow that checks each repo's `AGENTS.md` shared block against the sha256 of the block in `conventions/agents.md`; plus any shared-workflow change L2 requests (Postgres service for `CK_TEST_DATABASE_REQUIRED`).

### Step 3: flows artifact cross-check (R1.1b)

Artifact (private): https://claude.ai/artifact/TE44nJbEz8dJsBzfSkxYKb. Update the same URL, no secrets in it.
- [ ] Azure tab vs `ck-infra/infra/azure/*.tf`, `infra/website/*.tf`, `bootstrap.sh` (compare with `origin/develop`; L2 is changing it), then read-only `az resource list -g` for `rg-ckhub-dev`, `rg-ckhub`, `rg-ckhub-tfstate`, `rg-kc-dash`. If `az` is not signed in, ask the user; do not work around it.
- [ ] Every Hub route in `routes.rs` appears; each coverage-matrix claim (unit, integration, qemu-smoke CK#, ck-vmtest, manual) is verified by finding the test; add the signed-settings flow (protocol v0.5.0, ck-os #79, ck-hub #74, ck-forge #68).
- [x] Gaps go to `sessions/parallel/L4-findings.md` and are handed to L1 to L3 as R3.5 input under their Requests.

### Step 4: ck-website

- [x] The licences panel is merged (#21). Check the site against `ck-os/docs/licences.md` and the image's source offer; list drift. Nothing else is open for the website; report if you find more.

### Step 5: plan upkeep

- [x] R5.5 roadmap ids are in `IMPLEMENTATION-PLAN.md`; add a "past mistakes" list in `sessions/pitfalls.md` (not code) from the review reports in `review/`.
- [x] Draft (do not decide) the written question to Canonical and the `RELEASE_PLEASE_TOKEN` steps for the user; update the licence draft status (#31) from `ck-os/docs/licences.md`.

### Step 6 (last, **waits for** L1 to L3 open PRs merged): cross-repo files

One small PR per repo from fresh `origin/develop`, one repo at a time, only these files:
- [x] (merged 2026-11-23: ck-protocol#43, ck-os#86, ck-forge#72, ck-hub#92, ck-infra#48, ck-website#23; house-rules caller workflow NOT added yet, see note) `AGENTS.md` (synced shared block + a "This repository" section moved from that repo's `CLAUDE.md`, keeping every existing rule: branches, no attribution lines, never log secrets), `CLAUDE.md` = `@AGENTS.md`, `.coderabbit.yaml` (`reviews.profile: assertive`, `auto_review.base_branches: ["^develop$"]`, `ignore_usernames: ["dependabot[bot]"]`, shellcheck and clippy on, `path_instructions` for `ck-os/image/**`, `ck-crypto/**`, `ck-hub/src/{accounts,enrol,signing,commands}.rs`, `ck-infra/infra/**`). Do ck-protocol, ck-os, ck-forge, ck-hub, ck-infra, ck-website.
- [x] (merged: ck-protocol#45, ck-os#88, ck-hub#93, ck-forge#75 root workspace; ck-forge `app/` workspace and ck-infra/ck-website have no change) R5.2 apply the workspace lints and profiles from the template (one repo, one PR, fix what `unsafe_code = "forbid"` and the lints flag, or report it to the owner).
- [x] (shell scripts instead of xtask: .github#31 templates, copied by ck-protocol#47, ck-os#89, ck-hub#94, ck-forge#76; ast-grep rules not done) R5.3 `cargo xtask` (or `tools/check-*`) per repo: `secrets` (scan what git would commit), `git-sources` (lockfile sources against the contest-keeper org); optional ast-grep rules.
Done when the house-rules check passes in all six repos. **Open:** the caller workflow (`templates/house-rules.yml`) is not in the repos, because it reads `.github` `main`, which lacks `conventions/agents.md` until the user promotes `develop` to `main`.

## May touch

The `.github` repo, ck-website, the workspace folder, the artifact. Other repos: only step 6 files.

## Needs the user

CodeRabbit GitHub app install (R5.4), `RELEASE_PLEASE_TOKEN`, approval of every deletion, Canonical question.

## Requests

- From L2: in `.github` `rust-ci.yml`, set `CK_TEST_DATABASE_REQUIRED=1` when `postgres: true`, so a missing test database fails instead of skipping. ck-hub runs its own `postgres-tests` job meanwhile; remove it once the shared workflow does this.
- From L2 (2026-11-23): ck-hub `CLAUDE.md` line 3 says "the Svelte dashboard". Verified false: there is no `package.json` or Svelte anywhere in ck-hub; the dashboard is plain JavaScript and CSS in `crates/ck-hub/src/page/{index.html,theme.js,app.js,app.css}`, served from `/assets/` (ck-hub#88; its policy no longer allows inline script or style). Replace "the Svelte dashboard" with "the web dashboard". `routes.rs`, `store.rs` and `postgres_store.rs` are now directories (`routes/`, `store/`, `postgres_store/`) if CLAUDE.md names them.
