# L3: ck-forge and ck-protocol

Read `README.md` in this folder first (rules, including rule 11: implement each item, open the PR, wait for green CI and squash-merge it yourself). You own **ck-forge** and **ck-protocol** (all its crates) completely. Do not edit other
repos. Plan source: `sessions/plan-review-and-craft.md` and `lane-B-protocol-forge-agent.md` (read it for item history).

## Checklist (in order)

### Step 1: R2.15 sensitive Bearer headers (small, first)

- [x] `crates/ck-admin-core/src/hub.rs` builds `Bearer <token>` in many places; some call `set_sensitive(true)` (ck-forge PR #69)
  (about lines 356, 408, 497, 552), others are plain (about lines 148, 316, 741 on 2026-11-23; re-check with `rg -n "Bearer"`).
  Add one shared helper that builds the header and marks it sensitive; use it everywhere in ck-forge (core, CLI, `app/src`).
- [x] Test that a header built by the helper `is_sensitive()`; guard against new plain uses if simple (a clippy `disallowed-methods` entry or a test). (ck-forge PR #69)
- [x] Check tokens never reach logs, error text, `Debug` output or the progress JSON; test one error path. (ck-forge PR #69: Debug and error text tested; progress JSON not separately tested)

### Step 2: protocol housekeeping and wire safety

- [x] `version.txt` says 0.4.0 but v0.5.0 is tagged: find out why (release-please config) and fix if it is ours; report if it is by design. (already 0.5.0 on develop, release PR #41; nothing to fix)
- [x] If L1 asks for it under Requests: remove `ESP_CONFIG_FILE` and the ESP settings location from ck-protocol and ck-forge (the stick no longer reads ESP settings after L1 step 2). (no request from L1; left alone)
- [x] #47 chrono: confirm none is left in ck-protocol and ck-forge (`Cargo.lock`, `deny.toml` ignore RUSTSEC-2026-0543); remove it keeping the JSON wire format byte-identical (add an old-fixture parse test). (no chrono in ck-protocol or ck-forge; nothing to remove)
- [x] **R3.5** hostile-input sweeps for ck-protocol JSON and validation, `ck-image` GPT, FAT and CKCONFIG (proptest, shared `PROPTEST_CASES`). (ck-protocol PR #42)

### Step 3: Forge product items still open (lane B list)

- [ ] (Mac session: Forge client side already merged as #61 and `mint_enrolment_code`; the packaged-VM test needs Windows/Hyper-V + ck-vmtest, not here) **#24** one-time enrolment: the Forge client side, then a packaged-VM test against the local Hub (Hub side exists; coordinate fields with L2 via Requests).
- [ ] **#32 to #34** developer image creator: finish `ck-forge image create` on a fresh branch from `develop` (the old `fix/developer-image-provisioning` is merged as #62; re-check the open points in `lane-B` first), boot the result with `ck-vmtest` (PASS required), run the patch step on the Ubuntu image. This is developer-only (decision 1); keep it short, and drop what L1's R4.2 ICPC removal makes obsolete.
- [ ] **#27** Forge fetches `images.json`, downloads, verifies the hash, writes the USB: the code exists (`feat: add hub image downloads to forge`); mark done only after the user's G4 acceptance (below). Prepare everything else.

### Step 4: tests and cleanup (R3.3, R3.4, R4.3 to R4.5 for Forge and protocol)

- [x] (verified 2026-11-24: every other crate has one test binary; nothing left to move) **R3.3** consolidate integration tests into `tests/it/main.rs` for ck-admin-core, ck-image and the other crates with several binaries (move-only PR per crate). (ck-image: ck-protocol#46 merged. ck-admin-core: ck-forge#77 open, CI green, NOT merged: merge it first. Other crates not checked.)
- [x] (ck-protocol#48, ck-forge#78 merged; thin tests only, more may be cut later) **R3.4** judge and cut, one PR per crate, verdict lines in the PR (thin tests in `ck-crypto` are first candidates). Use `.github` `conventions/testing.md` (**waits for** L4 step 2; start with: keep a test only if it checks an invariant, a plausible breaking change, and nothing else covers it).
- [x] (ck-forge#70, ck-protocol#44 merged; `cargo machete` run 2026-11-24 in both repos: nothing unused) R4.3 remove wrapper pairs `configure_image_signed` and `build_thin_image`; the `Signer`/`Verifier` traits in `ck-crypto` only after grepping ck-os and ck-hub for uses (they cannot be edited here: ask under Requests if a repo still uses them); `cargo machete` in each workspace.
- [x] (ck-forge#79 merged; 2026-11-24 sweep: no history comments left, remaining long doc blocks are 7-12 lines of behaviour notes) R4.4 remove restating/history comments; shorten long doc blocks (`write.rs:75-97`, `thin.rs:1-12`). (ck-forge#73 shortened ck-writer docs; the rest of the sweep, and ck-protocol, not done.)
- [x] (ck-forge#71 merged) R4.5 split `ck-forge/src/main.rs` (927 lines) per subcommand.
- [x] (2026-11-24 review done in notes folder: one fix, ck-protocol#49; ck-forge hub.rs read for redirects/timeouts/https, no defect; ck-base-image not re-read) (ck-crypto re-read, no defect found; the rest still open; partial: `sessions/review-2026-11-23-ck-protocol.md` and `...-ck-forge.md` written, fix PRs not done) R4.1 review sweep (ck-crypto and ck-protocol validation first, then ck-writer and `ck-image`, then Forge); findings in `sessions/review-2026-11-<dd>-ck-protocol.md` and `...-ck-forge.md`, then small fix PRs.

### Step 5: egui Forge (R6), long-term, last

Start only after steps 1 to 4 are merged.
- [x] R6.1 (merged: #80, #81, #83, #84; every window command now has a core function and a CLI route) make the engine GUI-free: every one of the 24 Tauri commands (`app/src/main.rs:23-48`) gets a function in `ck-admin-core` and a `ck-forge` CLI subcommand; the elevated `ck-forge stick write` with its JSON progress file stays.
- [ ] (ck-forge#85 merged; #86, #87, #88 open, stacked; open rows in docs/egui-parity.md: write sticks, catalog/import, preview, bookmarks/Wi-Fi/recording, updater) R6.2 new `app-egui` crate (eframe) next to `app/`, screens in order: Hub sign-in, profile editor (contest, network, branding), image picker and download, write sticks, retire and re-key; keyring vault, updater and Windows elevation equivalents; build against a parity checklist made from the Svelte pages.
- [ ] R6.3 only after parity is tested on Windows **and the user agrees**: remove `app/`, `ui/` and their CI jobs.

## Status (2026-11-23, session stopped by the user)

Merged: ck-forge #69 to #76, ck-protocol #42 to #47. ck-forge#77 (R3.3 ck-admin-core) has since merged; no PR is open. R3.4 started but the stop left `L3-ck-protocol-4` broken (redo from fresh develop). R3.4 was started (local branches `refactor/cut-thin-tests`, ck-protocol worktree `L3-ck-protocol-4`, locked; check `git log` before reusing). Forge moved to org routes and `email` login (#74); the Hub drop of the old routes waits for a new Forge release (user's call). Untouched: R3.4 rest, Step 3 (#24, #32 to #34, #27), Step 5 (egui).

## May touch

All of ck-forge and ck-protocol.

## Needs the user

Writer verification on the Kingston, G4 acceptance for #27, hardware tests, approving R6.3 removal.

## Requests

- From L1 (ck-os): once ck-os PR "remove the ESP settings fallback" is merged, ck-os no longer reads `ck/config.json` on the EFI partition or imports `ESP_CONFIG_DIR` / `ESP_CONFIG_FILE`. Remove those constants from ck-protocol when ck-forge no longer writes the ESP copy, and make sure `ck-forge image build` writes only the CKCONFIG partition. Also: a release ck-agent now needs `/etc/ck/config-public-keys` in the image (ck-os PR 83), so images built from a base without that file will not apply settings.
- From L2 (2026-11-23): ck-hub still serves the unscoped "sole organization" routes (`/api/v1/sticks`, `/api/v1/sticks/{id}/...`, enrolment, commands) and accepts `{"username": ...}` as the login field, because ck-forge uses them (`ck-admin-core/src/hub.rs`). Please move ck-forge to the `/api/v1/orgs/{slug}/...` routes and to `{"email": ...}` for login, and tell L2 (or leave a line here) when that is merged. Only then can the Hub drop the old routes and the `username` alias (R4.3). Until then the Hub keeps both: removing them first would break ck-forge.
