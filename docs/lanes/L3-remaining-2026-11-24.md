# L3: what is left and exactly what blocks it (2026-11-24, Mac session)

Done in code, tested on this Mac: R6.1 (all 24 window commands have a ck-admin-core function and a ck-forge route), #27 code (Hub image list, download, SHA-256 check, import: ck-admin-core tests with a fake Hub; `ck-forge hub images|download`), #32 to #34 code (`ck-forge image create` over the catalog).
Open, with the blocker:
- #24 packaged-VM enrolment test: needs Windows with Hyper-V/QEMU, the packaged VM and the dev Hub (ck-vmtest). Forge side is merged (#61).
- #32 to #34 "boot with ck-vmtest PASS" and the patch step on the Ubuntu image: same, Windows + ck-vmtest + the Ubuntu base image.
- #27 "mark done": the user's G4 acceptance (organizer login on the dev Hub) and a real USB write on the Kingston.
- R6.2 write-sticks screen: Windows UAC elevation of `ck-forge stick write`, USB hardware. Other rows (catalog build, archive import, wallpaper preview, Wi-Fi, recording, Hub profiles, save in Forge folder, updater): see docs/egui-parity.md; Wi-Fi holds a secret, the updater and installer need the release pipeline.
- R6.3: the user's agreement plus the parity rows tested on Windows; checklist is in docs/egui-parity.md (ck-forge#89).
- ck-base-image/detect.rs still names an "ICPC contest reference image" fingerprint (detect.rs:21): kept, it identifies an old image by hash; remove if the user wants all ICPC wording gone.
