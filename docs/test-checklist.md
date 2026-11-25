# Test checklist: every manual test you have to run

One place for everything that needs a person, hardware, Windows, a login or a decision. Each item has steps and the expected result.
Tick a box only when you saw the expected result. Never write a Wi-Fi password, token or key into this file or into a chat.

Written 2026-11-25 from the lane files, the per-repo `docs/next-steps.md` files, `docs/testing-the-writer.md`, `docs/egui-parity.md`, `docs/user-checks.md`,
the ck-os review file and `sessions/hardware.md`. Items that depend on work that is not merged yet say **needs** and name the pull request or lane;
do them only after it is merged (`gh pr list --state merged` in that repository, or the repository's `docs/next-steps.md`).

## What you need

- 2 or 3 laptops, one without an Ethernet port (decision 11). USB sticks of at least 32 GB that you can erase (the image is 29,884,416,000 bytes).
- A Windows PC with administrator rights; for packaged-VM tests also Hyper-V or QEMU. A Mac for the macOS tests.
- An organizer login on the dev Hub (`https://hub.dev.contestkeeper.com`), and the dev website (`https://dev.contestkeeper.com`).
- The latest Forge preview build: GitHub Releases of `contest-keeper/ck-forge`, tag `dev` ("Preview build 0.1.0-preview.N"; unsigned, so Windows SmartScreen warns).
- Where to write results: the per-laptop table in `sessions/hardware.md` (copy it once per laptop and stick), and a note per item below.

## ck-os (the image)

State today: Debian profile is built and passes the QEMU boot modes in CI; Mint and Fedora profiles, the read-only root with dm-verity and the signed UKI are
lane L9 and are **not merged** at the time of writing; the agent fixes are lane L8 (open pull requests ck-os #119, #121 to #126).

### OS-1 Debian image on real laptops: UEFI, Secure Boot, legacy BIOS

For each laptop (fill the table in `sessions/hardware.md`):

1. In Forge, make a contest profile and write a stick (see F-1 or F-2), or write a published image with `ck-forge stick write` (hardware.md, "Make a test stick").
2. Boot the stick from the laptop's boot menu key in each mode that the laptop has:
   a. UEFI with Secure Boot **off**;
   b. UEFI with Secure Boot **on** (the image boots through the distribution's shim; with the signed UKI of OS-4 a one-time MOK enrolment is needed first);
   c. legacy BIOS / CSM, if the laptop has it.
3. In each mode check: it reaches the XFCE desktop with the `contestant` account logged in without a password; the GRUB menu cannot be edited without a password;
   the computer name equals the one in the profile; wallpaper, bookmarks, home page and welcome text from the profile are present.
4. Network: Wi-Fi connects to the contest network (adapter and driver work); the network lock lets the contestant reach only the allowed sites (open another site in Firefox: refused).
5. Recording: if the profile sets recording, it runs and the file grows (`ls -l` in the recordings folder as admin).
6. Touchpad, keyboard layout, screen resolution, brightness keys, suspend. Time from power on to desktop (`systemd-analyze`).

Expected: boots in every mode the laptop offers, all of step 3 to 5 true. Record each "no" and any surprise in the table; surprises that change other work go to the lane owner.

- [ ] Laptop 1: UEFI, Secure Boot off, Secure Boot on, BIOS
- [ ] Laptop 2: same
- [ ] Laptop 3 (no Ethernet): same, Wi-Fi is the only network

### OS-2 Linux Mint on real laptops (#29)

**Needs** L9: the Mint profile merged and an image published (see `ck-os/docs/image-profiles.md` and the result in `docs/testing.md` for the QEMU boot).
Steps and expected result are OS-1, with the Mint image. Also check: the image carries no Ubuntu or Mint branding that the licence draft forbids, and
the source offer text (`SOURCE-OFFER.txt`, built from the profile's `SOURCE-OFFER.txt.in`) is on the image; `docs/licences.md` says where.

- [ ] Laptop 1 / 2 / 3: UEFI, Secure Boot on, BIOS

### OS-3 Fedora on real laptops (#29)

**Needs** L9: the Fedora profile merged and an image published. Steps and expected result are OS-1, with the Fedora image. Fedora uses SELinux: check
`getenforce` says Enforcing and that nothing important is denied (`journalctl -b | grep -i avc`).

- [ ] Laptop 1 / 2 / 3: UEFI, Secure Boot on, BIOS

### OS-4 Secure Boot with MOK enrolment (signed UKI)

**Needs** L9 R2.2c (signed UKI, organizer MOK enrolment doc) merged, and the signing key added by you to the CI secrets. Follow the enrolment doc in `ck-os/docs`
(`verity-and-signed-uki.md` describes the design). On a laptop with Secure Boot **on**:

1. Boot the stick once. The firmware or shim shows a "Security violation" or the MOK manager (blue screen): this is expected before enrolment.
2. Enrol the key: from the stick's admin shell run `mokutil --import <cert>` (the doc gives the exact path), set a one-time password, reboot, choose "Enroll MOK" in the blue
   screen, enter the password, reboot.
3. Boot the stick again. Expected: it boots to the desktop; `mokutil --list-enrolled` shows the Contest Keeper key; `cat /proc/cmdline` contains `roothash=`;
   `cat /sys/kernel/security/lockdown` shows `[integrity]`.
4. Repeat on a second laptop to measure the per-laptop cost (decision 15 accepts one enrolment per laptop). Note how long it took.

- [ ] Laptop 1 enrolled and boots; Laptop 2 enrolled and boots

### OS-5 dm-verity tamper test

**Needs** L9 R2.2a to R2.2d merged (read-only root, verity hash tree, signed UKI, qemu-smoke tamper test). On a laptop with the key enrolled (OS-4):

1. Write the protected image to a stick. Boot it: desktop as in OS-1. `lsblk` and `veritysetup status` show the root as a verity device.
2. Tamper with it: plug the stick into another Linux machine, flip one byte inside the root partition (for example
   `printf '\x00' | sudo dd of=/dev/sdX2 bs=1 seek=1048576 conv=notrunc`; the doc gives the exact partition). Do not touch the boot partition.
3. Boot the stick again. Expected: it does **not** reach the desktop; the console shows a dm-verity corruption or "verification failed" error and the boot stops (or reboots).
4. Write a fresh copy and confirm it boots again. Then write the **unprotected fallback** image (the build keeps the ability) and confirm it boots on a laptop without the key.

- [ ] Untouched protected stick boots; tampered stick refuses; fallback image boots

### OS-6 Bluetooth option takes effect

**Needs** nothing new (the agent applies it at boot with `rfkill`). On a laptop with Bluetooth:

1. In Forge, profile editor: tick **Block Bluetooth**, write a stick, boot it.
2. As admin run `rfkill list bluetooth`. Expected: `Soft blocked: yes`; the desktop's Bluetooth applet cannot turn it on.
3. Write a second stick with the option **off** (the default). Expected: `Soft blocked: no` and Bluetooth can be used.

- [ ] Blocked when set; free when not set

### OS-7 USB-storage option takes effect

**Needs** ck-os #115 (merged) and Forge #96 (merged). On any laptop:

1. In Forge, profile editor: tick **Block USB storage**, write a stick, boot it.
2. Plug a second USB drive (not the stick). Expected: no new disk appears (`lsblk` shows only the stick; `dmesg | tail` shows the mass-storage interface was not bound); the desktop shows
   nothing. The stick itself keeps working (open Firefox, save a file in the contestant's home).
3. Write a stick with the option **off**. Expected: the second drive shows up in `lsblk` (it is still not mounted for the contestant: udisks is not installed).
4. Unplug and replug the stick's own port: the system stays up.

- [ ] Blocked when set; visible when not set; boot stick unaffected

### OS-8 Review items that need a booted image

From `sessions/review-2026-11-24-ck-os.md` (marker `needs booted image`; re-read that file before running, L8 and L9 add rows):

- I9: with the **Ubuntu** profile, log in as contestant: the restricted desktop menu is in force (the menu files, not the full XFCE menu). Expected: the menu files under `/etc/xdg/xdg-xubuntu` are read by the `xfce` session.
- I6: `systemctl status ck-selftest` shows the unit ran at boot (the journal shows no ordering-cycle message already).
- Any row L8 or L9 marks `needs booted image` (apply order, revert of removed settings A5, capture server, squid CA renewal CK58).

- [ ] I9, I6 and every row that the review file marks `needs booted image`

### OS-9 First-boot enrolment of a stick against the dev Hub, on a real laptop

1. In Forge sign in to the dev Hub, make a profile with enrolment (the default), write the stick (F-1 or F-2).
2. Boot on a laptop with network. Within one hour (the token life) the stick enrols. Expected: the stick shows in the dev Hub dashboard (Sticks) as online; its heartbeat updates.
3. In the dashboard send a **message** command; expected: it appears on the stick's desktop within a minute. Send **lock screen**: the screen locks; unlock from the dashboard.
4. Retire the stick's credential in the dashboard (or in Forge, Sticks page): expected the stick can no longer report.

- [ ] enrol, message, lock/unlock, retire

### OS-10 Settings signed by the Hub are applied; unsigned are refused

1. Needs the dev Hub to hold the signing key (H-6). In Forge (signed in) write a stick with settings (a changed home page and the USB option). Boot: settings applied (home page changed).
2. Then, as admin on the booted stick, copy a hand-edited `config.json` (no valid signature) onto the `ck-config` partition from another computer and reboot: expected the stick ignores it
   (the home page stays the old one; `journalctl -u ck-config` or the agent log says the file was rejected). The CI check `CK45` covers the refusal in QEMU; this proves it on hardware.

- [ ] signed applied; hand-edited refused

## ck-forge (the writer and the egui window)

State today: the egui window has sign-in, profile editor, image download, sticks page and the write page on macOS; lane L10 adds Wi-Fi and recording (ck-forge #98 open),
Hub profiles, image picker, the Windows elevated write, updater and macOS extras. The old Tauri window (`app/`, `ui/`) stays until R6.3.

### F-1 Real stick write on macOS

From `docs/testing-the-writer.md`, "Manual test with a real stick on macOS". Use a stick you can erase, plugged in directly (no hub).

1. `cargo build --release -p ck-forge`, then `target/release/ck-forge stick list`: your stick shows `USB` and `[can be written]`; internal disks and the boot disk say `refused`. If your stick says refused, stop and read why.
2. Without `sudo`: `ck-forge stick write stick.img --disk N` stops with "run the command again with sudo" and the stick is still mounted in Finder.
3. With `sudo`: type `erase N` when asked. Bars show writing, then checking; ends with "Read back and checked, SHA-256 ...".
4. `diskutil list` shows the stick with the image's partitions (macOS may offer to eject: click Ignore).
5. Pull the stick, plug it in, write again: quicker (unchanged blocks are skipped), same hash.
6. Try `--disk` with your internal disk's number: refused, nothing written.
7. Boot a laptop from the stick (OS-1).
8. Optional, safe: `tools/test-writer-disk-image.sh` (a scratch disk with `hdiutil`, no `sudo`).

- [ ] steps 1 to 7 as expected

### F-2 Real stick write on Windows (Kingston stick)

From `docs/testing-the-writer.md`. Administrator PowerShell; unplug other USB drives first.

1. Optional first: `tools/test-writer-vhd.ps1` (a VHD target, needs free space equal to the image, about 30 GB): writes and verifies without a stick.
2. `ck-forge stick list`: check the number, size and name are the stick and nothing else; the system disk and internal drives show as refused.
3. `ck-forge stick write stick.img --disk N`; type `erase N`. Expected: writes, then "Read back and checked" with SHA-256, no error. Unplug, replug, write again: faster.
4. Try your data disk's number: refused. Try a stick smaller than the image: "use a larger stick".
5. Boot a laptop from it (OS-1). This also closes the writer check for #27.

- [ ] steps 1 to 5 as expected

### F-3 Windows parity rows of the egui window (`docs/egui-parity.md`)

Run on a Windows PC against the dev Hub, with the newest preview build of the egui window. For every row, use the screen, then mark the row `[x]` in `docs/egui-parity.md` (a small PR; a session can do it
when you say the row passed). Rows and what to do:

| Row | Steps | Expected |
|---|---|---|
| Hub sign-in | Sign in with the organizer login, close and reopen the window, sign out | Browser opens, window shows the signed-in user, stays signed in after restart, sign-out clears it. **Needs** nothing new. |
| Profile editor | New profile, fill name, contest, team, site, computer name, USB label, server, sites, home page, welcome text, bookmarks, Bluetooth, USB storage; save; reopen; check | Values round trip; the check reports no errors |
| Wi-Fi and recording fields | **Needs** ck-forge #98 (and L10 Wi-Fi/recording rows). Enter an SSID and a password; set recording on and a contest end | The password field is masked, never appears in the saved profile in clear text beyond what the format requires, never in a log or the window title; the settings round trip |
| Save in the Forge folder | **Needs** L10. Save a profile "in the Forge folder" | File is under the Forge app data folder and appears in the list |
| Wallpaper add and preview | **Needs** L10 preview. Add a picture | Preview shows it; a non-picture is refused |
| Hub profiles | **Needs** L10. List, load, save a profile to the dev Hub; delete it | Round trip; another organizer's profile is not offered |
| Image picker: catalog, create, import | **Needs** L10. Pick from the catalog; create a base image; import a downloaded `.img.zst` with its SHA-256 | Progress shown; the imported file verifies; bad hash is refused |
| Image picker: Hub download | Download the current image from the dev Hub | Download completes, checksum matches |
| Write sticks | **Needs** L10 (UAC elevation, JSON progress file, cancel). Pick a stick, write | A UAC prompt appears, progress moves, **Cancel** stops cleanly and leaves the stick untouched or marked incomplete, a full run ends with the SHA-256 check; refuses the system disk |
| Retire and re-key | Open Sticks, retire a stick key | The stick is listed as retired in the dev Hub |
| Updater | see F-4 | |

- [ ] every row above passed (then decide R6.3, F-6)

### F-4 Updater and installer: install over an older release

**Needs** L10 updater and the installer step in the release workflow.

1. Install an older preview build (or release) with its installer on Windows. Run it once.
2. Publish or wait for a newer preview (every merge to `develop` builds one), or use the newer build you have.
3. In the old build, check for updates. Expected: it finds the newer one, downloads it, verifies it, and restarts into it. `About` shows the new version.
4. Run the new installer over the old install: expected it replaces it in place (same install folder, settings and sign-in kept).
5. Note whether SmartScreen warns (signing is a decision, see D-4).

- [ ] updater found, downloaded, restarted; installer over old install keeps settings

### F-5 Packaged-VM tests with `ck-vmtest` (#24, #32, #33, #34)

On Windows with Hyper-V or QEMU. See `ck-os/docs/testing.md` and `ck-os/docs/image-creator.md`.

1. #32: `ck-forge image create` (developer path) a full base image from the catalog. Expected: it ends without error and writes a base image file.
2. #33: `cargo run -p ck-vmtest -- --image <result.img> --out <folder>`. Expected: `PASS` (GPT, EFI system partition, boots, selftest ok).
3. #24: enrolment against the dev Hub: build an image for a profile with enrolment, boot it with `ck-vmtest` (network on). Expected: the stick appears in the dev Hub as enrolled.
4. #34: run the patch step on the Ubuntu base image (WSL or in-guest, per `image-creator.md`). Expected: patched image boots with `ck-vmtest` PASS.
5. Tell a session the result so #24, #32 to #34 are ticked.

- [ ] #32 built; #33 PASS; #24 enrolled; #34 patched and PASS

### F-6 Decision R6.3: remove the old Tauri window (`app/`, `ui/`)

Only after F-3 and F-4 are fully ticked **and** you agree (you said "not yet" on 2026-11-25). Then tell a session: "do R6.3". It removes `app/`, `ui/`, their CI jobs and the Svelte and Tauri words in one PR;
you then drop `ui / check` from the required checks of ck-forge (branch protection is yours to edit).

- [ ] parity done and agreed; protection updated after the PR

### F-7 macOS extras

**Needs** L10: Hub session in the Keychain, QEMU firmware lookup for `image create`, and a release carrying `keeper-forge-macos-universal.zip`.

1. On a Mac: sign in to the Hub in the egui window, quit, reopen: still signed in; the session is in Keychain (`security find-generic-password -s <service from the docs>`), not in a plain file.
2. Download and unzip `keeper-forge-macos-universal.zip`, open the app (unsigned: right-click, Open). Expected: it starts on both Intel and Apple Silicon if you have both.
3. `ck-forge image create` finds the QEMU firmware files without extra flags (Homebrew QEMU).

- [ ] Keychain session; zip runs; image create finds firmware

## ck-hub (dev Hub and Azure)

State today: the Hub code is merged and tested; the read-only Azure checks passed on 2026-11-24. Names come from `ck-infra/docs/azure.md` and `ck-hub/docs/user-checks.md`.

### H-1 G4 acceptance (#27)

1. Log in as organizer on `https://hub.dev.contestkeeper.com`, open **Downloads**.
2. Open the manifest link and the source-offer link of the current image: `/download/image/{id}` and `/download/image/{id}/source-offer`. Expected: both answer for a logged-in organizer.
3. Start the image download. Expected: it starts (redirect to a signed blob link) and finishes with the checksum shown on the page.
4. Log out and open the same address: expected a login prompt or 401, not the file.
5. Tick #27 (tell a session).

- [ ] steps 1 to 4

### H-2 Recordings round trip (real Azure)

From `docs/user-checks.md` section 1. Needs one enrolled stick (OS-9) or a tool that signs like a stick.

1. Log in, open the stick, start a recording on it. The stick posts `POST /api/v1/recordings` and PUTs chunks.
2. In the dashboard open the recording: it plays. Open the link in a new private window to prove the link, not your session, grants access; it must stop working after its short lifetime.
3. Delete the recording in the dashboard and confirm the blob is gone:
   `az storage blob list --account-name ckhubdevdl10lhs --container-name recordings --auth-mode login --query "[].name" -o tsv`.
4. Retention: leave a recording older than the Hub's retention period and confirm it is removed.
5. A `403 AuthorizationPermissionMismatch` or `AuthenticationFailed` answer: see `docs/storage-identity.md` step 5 and report the log line.

- [ ] play, delete, retention

### H-3 Capacity run on Azure (approved, cap 5 USD)

From `ck-hub/docs/next-steps.md` section 2 and `docs/user-checks.md` section 3. Approved: 50 and 100 sticks, the current 0.25 vCPU container, **hard cap 5 USD**, no scale-up.

1. Baseline first: record the accumulated cost of `rg-ckhub-dev` for the month (portal Cost Management, scope `rg-ckhub-dev`).
2. `cargo run --release --example loadsim -- registry --sticks 50 --out registry.json`; push it into a **throwaway organization** of the dev Hub.
3. `cargo run --release --example loadsim -- run --hub https://hub.dev.contestkeeper.com --registry registry.json --minutes 30`. Watch Container App CPU (Metrics) at about 15 minutes. Then the same for 100 sticks.
   Never more than 2.5 hours in total; stop with Ctrl+C when the time is up; stop at once if run cost plus 1 USD would pass 5 USD (check before each level, at 15 minutes and after each level).
4. Put the tables into `ck-hub/docs/capacity.md`. If CPU is the limit, stop and ask before raising `cpu` in ck-infra.
5. Archive and delete the throwaway sticks; read the cost again after cleanup and again a day later; note both in `docs/capacity.md`.

- [ ] 50 sticks run, 100 sticks run, numbers recorded, cost reconciled under 5 USD

### H-4 Dashboard walkthrough (the page script has no automated browser test)

On the dev Hub as organizer, then as viewer and as staff: sign up with a new email (needs mail, see H-5), accept an invitation, create an organization (staff), list and archive/restore/delete a stick,
look at network history and submissions of a stick, view a recording, send a command, edit a profile. Test on a narrow window (phone width) and in the dark theme.
Expected: no layout break (#99 fixed narrow screens), unknown addresses show the 404 page, viewers cannot change anything.

- [ ] organizer, viewer and staff paths

### H-5 Real mail through Azure Communication Services (only if the dev mail domain is verified)

1. Register a new account on the dev Hub with an address you own. Expected: a verification mail arrives from `noreply@mail.dev.contestkeeper.com`; the link verifies.
2. Use "forgot password": a reset mail arrives; the link sets a new password; the old one stops working.
3. Invite a second address to an organization: the invitation mail arrives; accepting works.

- [ ] verify, reset, invitation mails

### H-6 Dev Hub holds the settings signing key

1. Logged in as organizer: `GET /api/v1/config-signing-key` (open it in the browser while logged in, or `curl` with the session). Expected: 200 with a public key. `404` means the Hub has no `CK_CONFIG_SIGNING_KEY`
   (set the GitHub secret in ck-infra and let the pipeline deploy; sessions cannot).
2. The public key equals the one baked into the image (`/etc/ck/config-public-keys` on a booted stick).

- [ ] key present and matches the image

### H-7 Azure state versus Terraform (read-only)

Run the listing in `sessions/azure-tab-vs-terraform-2026-11-25.md` from an account that can read the Contest Keeper subscription, and compare. Share the flows artifact (`TE44nJbEz8dJsBzfSkxYKb`) with the account Claude Code uses, or paste the Azure tab.

- [ ] compared; differences reported

## ck-website

### W-1 Download buttons

1. Open `https://dev.contestkeeper.com` (develop channel): the Windows button downloads the installer through the Hub (`/download/forge`); the page says "Develop build ...".
2. Production channel (GitHub Pages, after the next promotion): the Windows button opens `.../releases/latest/download/keeper-forge-setup-x64.exe`.
3. **macOS button needs a release that carries `keeper-forge-macos-universal.zip`.** Until a release has that asset the macOS button deliberately opens the releases page (the develop channel has none). After L10 builds one and you cut a numbered release (D-3):
   open the site: the macOS button downloads `.../releases/latest/download/keeper-forge-macos-universal.zip`.
4. Licences panel: opens, closes with Escape and an outside click (#26).
5. The page on a phone width and in the dark/old look toggle.

- [ ] Windows ok; macOS ok after a release with the zip

## Decisions and tasks that are yours

### D-1 Licence approval (#31) and the Canonical question

1. Read `ck-os/docs/licences.md` (still a draft). Approve it or write what to change. The image must carry no Ubuntu trademarks unless Canonical approves.
2. Read `sessions/canonical-question.md`, send it to the trademark contact listed on the Canonical IP policy page (check the address that day), and keep the answer as `docs/canonical-answer.md`.
3. Tell a session the outcome: it then updates the image branding and the website text (the Mint and Fedora wording depends on it).

- [ ] licences approved; question sent; answer filed

### D-2 dm-verity design

Approved on 2026-11-25 (decision 15). Remaining for you: add the signing key to the CI secrets for the signed UKI (L9 R2.2c) and keep a copy of it safe; this is the root of trust.

- [ ] signing key in CI secrets and backed up

### D-3 Numbered Forge release

Pre-releases build on every merge to `develop`. When F-1 to F-5 pass: cut a numbered release (release-please opens the release PR; you merge it). Then check W-1 step 3.

- [ ] release cut

### D-4 Code signing

Windows: a code-signing certificate for the installer (SmartScreen warns on unsigned builds; the release workflow's signing step is guarded and waits for your certificate). macOS: Developer ID signing and notarisation.
Decide whether to buy them before the first public release.

- [ ] decided: Windows certificate; macOS signing and notarisation

### D-5 Housekeeping deletions

Read `sessions/housekeeping-2026-11-25.md`, approve or strike lines in "Proposed deletions", then tell a session (or run the commands yourself). Nothing is deleted without you.

- [ ] approved list

### D-6 Azure decisions

From `sessions/azure-tab-vs-terraform-2026-11-25.md`: register `Microsoft.Communication` in `bootstrap.sh`; decide how production gets a Container Apps environment (shared one or quota increase) and its domain and mail;
decide blob soft delete on the recordings account.

- [ ] decided

## Not on this list (already done by sessions)

Everything that runs without hardware, Windows or your login is covered by CI: unit and integration tests in every repo, the QEMU boot modes (UEFI, Secure Boot, BIOS) and the CK checks on the Debian and Ubuntu profiles,
the Hub route tests (`sessions/hub-route-coverage-2026-11-25.md`), and the read-only Azure checks of 2026-11-24.
