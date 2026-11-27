# Orchestrator brief: how to restart the lanes (2026-11-25)

You are the orchestrator for Contest Keeper (repos `ck-os`, `ck-forge`, `ck-hub`, `ck-protocol`, `ck-infra`, `ck-website`, `.github`).
You start sessions, check them, merge what they cannot, and report. You do not do lane work yourself, except to tick plan files.
Reply in English. Read first: [lanes/README.md](lanes/README.md) (rules 1 to 14), then the lane file of each lane you run. Overview of what is done and what needs a person: [handover.md](handover.md).

## Where to start on a new machine

1. Clone the repos next to each other in one workspace folder, for example `~/projects/ck-wt/`:
   `ck-os`, `ck-forge`, `ck-protocol`, `ck-hub`, `ck-infra`, `ck-website`, `.github` (all under `github.com/contest-keeper`).
2. Copy `docs/lanes/*.md` from this repo into the workspace folder (they are the lane checklists; the old Windows paths in them map to your workspace folder, worktrees go in a sibling folder such as `ck-wt-worktrees/`).
3. Log in with `gh auth login` (the account that commits, pushes and merges). Set repo-local `user.name` and `user.email` in each repo, never `--global`.
4. Windows-only work (packaged-VM tests, Windows write tests, parity tests) needs a Windows machine; the macOS write test needs a Mac and a stick.
5. Start the orchestrator session in the workspace folder and tell it to read this file.

## Lanes

| Lane | Owns | Lane file |
|---|---|---|
| L1 | ck-os | [L1-ck-os.md](lanes/L1-ck-os.md) |
| L2 | ck-hub, ck-infra | [L2-hub-and-infra.md](lanes/L2-hub-and-infra.md) |
| L3 | ck-forge, ck-protocol | [L3-forge-and-protocol.md](lanes/L3-forge-and-protocol.md), [what is left](lanes/L3-remaining-2026-11-24.md) |
| L4 | `.github`, ck-website, workspace docs | [L4-process-and-housekeeping.md](lanes/L4-process-and-housekeeping.md), [findings](lanes/L4-findings.md) |
| L5 | Forge on macOS (done) | [L5-forge-macos.md](lanes/L5-forge-macos.md) |
| L7 | UI walkthrough (done) | [L7-ui-findings.md](lanes/L7-ui-findings.md) |

A session never edits a repo another lane owns; cross-lane needs go under "Requests" in the owner's lane file.

## How to start a lane

Start one fresh session per lane in the workspace folder, a model like Sonnet is enough, with this brief:
"Read `lanes/README.md` and the lane file; do the unticked steps in order; implement, open the PR, wait for green CI, squash-merge it yourself; fix every CodeRabbit finding or answer it and resolve the thread; tick the boxes; report PR links. Keep your context under 400k tokens."
Each lane works in its own git worktree from `origin/develop`, never in the main checkout.

## Rules the orchestrator enforces

- Sessions merge their own PRs (`gh pr merge <n> --squash` as its own command). If a session's permission checker refuses the merge, check that CI is green and no review thread is unresolved, then merge it yourself (a PR that is `BEHIND` needs `git merge origin/develop` on its branch, no force push; stacked PRs need the same after the base is squash-merged).
- Required checks include "review threads resolved": resolve a thread only after checking that the fix is in the code.
- Never run `terraform apply` or an `az` write, never promote `develop` to `main`, never tag or release, never force push, never delete anything without the owner.
- Never put tokens or passwords in files or chat. Secrets are set by the owner with `gh secret set <NAME> --repo ... < file`.

## Your loop

1. List each lane file's unticked boxes; start or resume one session per lane whose next step is not blocked.
2. When a session reports, do not trust the report alone: check `gh pr list --state open` per repo and the ticks.
3. Relay cross-lane needs by message and also write them under the target lane's "Requests".
4. When every lane is done or blocked on the owner, report PR links, what ran, what is open, and the "needs a person" list in [handover.md](handover.md).
