# Handover (2026-11-25)

Where the Contest Keeper work stands and what to do next. Per-repo details are in `docs/next-steps.md` of `ck-os`, `ck-forge` and `ck-hub`.

## Plan to 100% (added 2026-11-25)

The remaining implementation work runs as lanes L8 (ck-os crates), L9 (ck-os image, Mint and Fedora profiles, dm-verity, approved as decision 15), L10 (ck-forge egui, updater, macOS extras) and L11 (process and the final test checklist). Lane files are in the workspace `sessions/parallel/`. Testing starts from [test-checklist.md](test-checklist.md); items that wait for a lane say which one.

## What is done

- Lanes L1 to L7 ran in parallel: ck-os, hub and infra, forge and protocol, process, Forge on macOS, the Hub images page, and a layout walkthrough of Hub and website.
- Roughly 87 percent of the lane checklists are ticked. Everything that can be built and tested without hardware, Windows or a decision is merged into `develop`.
- `.github` `develop` is promoted to `main`; the `house-rules` caller workflow is in all repos.
- CodeRabbit is installed on the organization (all repositories); `RELEASE_PLEASE_TOKEN` is set on every repo.
- Forge runs on Windows and macOS (CLI with `sudo` on macOS; egui window).
- USB storage and Bluetooth blocking are options set from Forge, off by default.

## What needs a person

Everything is listed with steps and expected results in [test-checklist.md](test-checklist.md). Summary:

| Item | Repo | Checklist |
|---|---|---|
| Debian, Mint, Fedora boots on real laptops (UEFI, Secure Boot, BIOS) | ck-os | OS-1 to OS-3 (Mint and Fedora after lane L9 merges the profiles) |
| Secure Boot with MOK enrolment, dm-verity tamper test | ck-os | OS-4, OS-5 (after L9 merges R2.2a to R2.2d); add the signing key to the CI secrets (D-2) |
| Bluetooth and USB-storage options take effect, items that need a booted image | ck-os | OS-6 to OS-8 |
| Real stick write on macOS and Windows | ck-forge | F-1, F-2 |
| Windows parity rows, updater install-over, then decide R6.3 | ck-forge | F-3, F-4, F-6 |
| Packaged-VM tests (#24, #32 to #34) | ck-forge | F-5 |
| G4 acceptance, recordings round trip, capacity run (cap 5 USD) | ck-hub | H-1 to H-3 |
| Dev Hub signing key and Azure state against Terraform | ck-hub, ck-infra | H-6, H-7 |
| macOS download button (needs a release carrying `keeper-forge-macos-universal.zip`) | ck-website | W-1 |
| Licence approval (#31), Canonical question, numbered release, code signing, housekeeping deletions, Azure decisions | all | D-1 to D-6 |

## Reports written by the process lane (2026-11-25)

In the workspace folder `sessions/` (not a git repository): `housekeeping-2026-11-25.md` (proposed deletions, nothing deleted),
`azure-tab-vs-terraform-2026-11-25.md` (the flows artifact and live Azure could not be read from the session) and
`hub-route-coverage-2026-11-25.md` (every Hub route against its tests, gaps, and the signed-settings flow).

## Items for the process lane (need the Windows workspace files)

These need `sessions/`, `IMPLEMENTATION-PLAN.md`, `DECISIONS.md` and the flows artifact from the Windows machine:
- apply the housekeeping deletions you approve (the audit is `sessions/housekeeping-2026-11-25.md`),
- keep the plan ticks true as the lanes merge,
- the Azure tab comparison needs the flows artifact shared with the Claude Code account and an `az` login on the Contest Keeper subscription.

## Working rules that were used

- Branch from `develop`, small PRs, squash into `develop`, titles `type: lowercase imperative subject`, no tool attribution lines in commits or PRs.
- Run `cargo fmt --all`, `cargo clippy --all-targets -- -D warnings` and the tests before every commit.
- Fix every CodeRabbit finding, or answer it with a reason, and resolve the thread; the required check "review threads resolved" blocks the merge otherwise.
- Never promote `develop` to `main`, tag, release, `terraform apply` or run an `az` write without the owner.
