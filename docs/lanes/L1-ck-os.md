# L1: ck-os (image, agent, security, profiles)

Read `README.md` in this folder first (rules, including rule 11: implement each item, open the PR, wait for green CI and squash-merge it yourself). You own **ck-os** completely (image, `tools/`, `docs/`, `crates/*`, workflows).
Do not edit other repos. Plan source: `sessions/plan-review-and-craft.md`; decisions 2, 3, 4, 5, 6, 7, 8, 10, 14 in `sessions/DECISIONS.md`.

## Checklist (in order)

### Step 1: R4.2 remove the ICPC path (several PRs)

`rg -i icpc` (not `target/`). On 2026-11-23 hits are in `image/patch/` (`rootfs/icpc/scripts/set_wallpaper.sh`,
`mask-units.txt`, `remove-files.txt`, `lib/image-tools.sh`, `lib/cloud-boot.sh`, `apply-to-image.sh`), `tools/patch-image.ps1`,
`tools/build-image`, `tests/test_image_patch.py`, `docs/icpc-image-checklist.md`, `docs/testing.md`, `docs/base-images.md`,
`crates/ck-agent/src/{self_test,hub_commands}.rs`, `crates/ck-agent-config/src/branding.rs` (+ tests),
`crates/ck-vmtest/src/check.rs`, `image/keeper/rootfs/etc/squid/squid.conf`, `image/keeper/rootfs/usr/local/bin/ck-wallpaper`.

- [x] Before deleting, find what Keeper OS and ck-forge's developer creator still use from `image/patch/` (the stock-Ubuntu
  cloud-boot patch, #76, may share `lib/`). Keep what is needed; ask the user if unclear.
- [x] PR: delete ICPC-only scripts, lists and Python tests.
- [x] PR: remove `/icpc/*` references in agent code and tests; the `loginctl` fallback in `hub_commands.rs` only if nothing else needs it.
- [x] PR: delete `docs/icpc-image-checklist.md`, clean `docs/testing.md` and `docs/base-images.md`, fix links.
- Never copy the old ICPC default hashes anywhere (decision 10).
Done when `rg -i icpc` hits only intentional history; checks pass; `keeper-image.yml` builds and qemu-smoke passes UEFI, Secure Boot and BIOS.

### Step 2: R2.1 finish the signed-settings gate

- [x] `keeper-image.yml`: a publishing build fails when `CK_CONFIG_PUBLIC_KEYS` is empty (today the step is
  `if: vars.CK_CONFIG_PUBLIC_KEYS != ''`). CI smoke builds keep working with a marked test key.
- [x] Agent: missing `/etc/ck/config-public-keys` is fatal in release builds.
- [x] Remove the ESP settings fallback (`crates/ck-agent-config/src/load.rs`: `EFI_FOLDER`, `Source::Esp`, `ESP_CONFIG_FILE`) and tests.
  If ck-protocol's `ESP_CONFIG_FILE` must go, write a line under L3 Requests (below, to the L3 owner via L3's file).
- [x] New qemu-smoke check (CK45 and up): the stick refuses unsigned settings; it must fail before the change.

### Step 3: R2.3 to R2.10 stick security (one PR each, each adds a qemu-smoke check that fails before and passes after)

- [x] R2.3 polkit denies NetworkManager connection add/edit for `contestant`; pin DNS; decide whether `nm-applet` stays (only switches Wi-Fi).
- [x] R2.4 blacklist `usb-storage`, Bluetooth and Thunderbolt in modprobe; udisks2 polkit deny; `/home` `0750`. (Merged in #90: Thunderbolt, udisks2, home modes; usb-storage/Bluetooth left for a user decision.)
- [x] R2.5 passwordless `ckreset` greeter login only after a Hub command or with the admin password.
- [x] R2.6 release kernel cmdline `lockdown=integrity` (or confidentiality), no `console=ttyS0` in published images (CI smoke only), GRUB without a menu.
- [x] R2.7 netlock: per-unit cgroup matching for `ck-uplink`, `systemd-timesyncd`, `squid` instead of `meta skuid root`.
- [x] R2.8 harden `ck-uplink` and `ck-config` units (DynamicUser or a `ck` user, `CapabilityBoundingSet`, `RestrictAddressFamilies`, `SystemCallFilter`).
- [x] R2.9 mount `/tmp` and `/home` `nosuid,nodev`; document the contestant-shell assumption in `docs/contestant-desktop-threats.md`.
- [x] R2.10 limit offline-readable secrets: per-stick CA key generated at first boot on the state partition; short credential windows.
  (The Hub-side "same stick from two places" flag is an L2 request.)
- [ ] R2.2 (design note merged in #97, not approved yet) dm-verity read-only root and signed UKI: **design spike first** (note in `docs/`, then ask the user before building); last in this step.

### Step 4: image profiles (#29) and remaining plan items

- [ ] #29 Linux Mint, then Fedora, profiles in `image/profiles/<family>`, one at a time, each with a real boot test in UEFI, Secure Boot and BIOS (Debian exists).
- [x] #45/#46 confirm `docs/base-images.md` describes the CI-built flow and the developer flow, `tools/build-image` passes shellcheck; tick if so.
- [x] #47 chrono: report whether any chrono is left in ck-os (`Cargo.lock`, `deny.toml` ignore of RUSTSEC-2026-0543) and remove it, keeping the JSON wire format identical.

### Step 5: tests and cleanup (R3.3, R3.4, R3.5, R4.3 to R4.5 for ck-os)

- [x] R3.3 consolidate each crate's integration tests into `tests/it/main.rs` (move-only PR per crate).
- [x] R3.4 judge and cut tests, one PR per crate, listing every removed test as verdict lines (`file:line name CUT C3`). Candidates: `ck-agent/tests/uplink.rs`. Use the testing standard L4 writes in `.github` `conventions/testing.md` once merged (**waits for** L4 step 2; start with the keep rule: invariant, plausible breaking change, only this test).
- [x] R3.5 add what the coverage gap list needs: hostile-input sweeps for the ICAP parser and the stick-side CKCONFIG reading (proptest with a shared `PROPTEST_CASES`), one test per Step 3 item.
- [x] R4.3 remove the orphan `crates/ck-agent-capture/`, wrapper pairs, unused dependencies (`cargo machete`).
- [x] R4.4 remove comments that restate code or describe history (`tools/build-image:2-3` header and others); keep "why" and guard comments.
- [ ] R4.1 review sweep of ck-os `image/` and `crates/`, every file read fully; findings in `sessions/review-2026-11-<dd>-ck-os.md`, then fix in small PRs.

## Status (2026-11-23, session stopped by the user)

Merged: ck-os #80 to #90 (R4.2, R2.1, R2.3, R2.4, lints, check scripts, AGENTS.md). Not committed: R2.5 work in worktree `D:\ck-wt\L1-ck-os` (branch `feat/allow-reset-login-on-organizer-action`; staged `ck-screen`, `ck-team-reset`, `tools/build-image`, `tools/qemu-smoke`, new `ck-reset-allow`). Its WSL before/after smoke run did not report; re-run it, check it fails before and passes after, then commit, PR, review, merge. Open for the user: whether to blacklist usb-storage and Bluetooth (left out of R2.4). Nothing else from Steps 3 to 5 is started.

## May touch

All of ck-os. Only the checklist boxes in this file and `IMPLEMENTATION-PLAN.md` for tracking.

## Needs the user

Real laptop boots (#5), licence approval (#31), approval of the dm-verity design.

## Requests

(none yet)

Notes (L1): Step 1 merged in ck-os PRs 80, 81, 82; local WSL build + qemu-smoke passed UEFI, Secure Boot and BIOS (login prompt) on develop after the merges. The keeper-image.yml workflow publishes when CK_IMAGE_PUBLISH_ENABLED=true, so it was not dispatched.
