# Handover (2026-11-25)

Where the Contest Keeper work stands and what to do next. Per-repo details are in `docs/next-steps.md` of `ck-os`, `ck-forge` and `ck-hub`.

## Plan to 100% (added 2026-11-25)

The remaining implementation work runs as lanes L8 (ck-os crates), L9 (ck-os image, Mint and Fedora profiles, dm-verity, approved as decision 15), L10 (ck-forge egui, updater, macOS extras) and L11 (process and the final test checklist). Lane files are in the workspace `sessions/parallel/`. After they merge, testing starts from `docs/test-checklist.md`.

## What is done

- Lanes L1 to L7 ran in parallel: ck-os, hub and infra, forge and protocol, process, Forge on macOS, the Hub images page, and a layout walkthrough of Hub and website.
- Roughly 87 percent of the lane checklists are ticked. Everything that can be built and tested without hardware, Windows or a decision is merged into `develop`.
- `.github` `develop` is promoted to `main`; the `house-rules` caller workflow is in all repos.
- CodeRabbit is installed on the organization (all repositories); `RELEASE_PLEASE_TOKEN` is set on every repo.
- Forge runs on Windows and macOS (CLI with `sudo` on macOS; egui window).
- USB storage and Bluetooth blocking are options set from Forge, off by default.

## What needs a person

| Item | Repo | Detail |
|---|---|---|
| Approve the dm-verity design | ck-os | `docs/verity-and-signed-uki.md` |
| Mint and Fedora boot tests (#29) | ck-os | UEFI, Secure Boot, BIOS on real hardware |
| Real stick write tests, G4 | ck-forge, ck-hub | macOS and Windows checklists in `ck-forge/docs/testing-the-writer.md` |
| Windows parity tests, then decide R6.3 | ck-forge | `docs/egui-parity.md` |
| Packaged-VM tests (#24, #32 to #34) | ck-forge | Windows with Hyper-V or QEMU, `ck-vmtest` |
| Capacity run, 50 to 100 sticks | ck-hub | approved, cap 5 USD, see `docs/user-checks.md` |
| Licence approval (#31), Canonical question | ck-os | `docs/licences.md` |
| Numbered Forge release, code signing | ck-forge | pre-releases build on every merge |

## Items for the process lane (need the Windows workspace files)

These need `sessions/`, `IMPLEMENTATION-PLAN.md`, `DECISIONS.md` and the flows artifact from the Windows machine:
- re-run the housekeeping audit and apply the approved deletions,
- keep the plan ticks true,
- Azure tab versus `ck-infra` Terraform, and the Hub route coverage matrix.

## Working rules that were used

- Branch from `develop`, small PRs, squash into `develop`, titles `type: lowercase imperative subject`, no tool attribution lines in commits or PRs.
- Run `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings` and the tests before every commit.
- Fix every CodeRabbit finding, or answer it with a reason, and resolve the thread; the required check "review threads resolved" blocks the merge otherwise.
- Never promote `develop` to `main`, tag, release, `terraform apply` or run an `az` write without the owner.
