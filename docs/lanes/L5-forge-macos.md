# L5: ck-forge on macOS

Rules: `README.md` in this folder (rule 11: each PR is opened, green, squash-merged by this session). Repo: **ck-forge** only.
Worktree: `/Users/robertpopa/projects/ck-wt-worktrees/L5-ck-forge`. Account: robbertpopa. No tool attribution in commits/PRs.
HARD RULES: never write to, erase, unmount or dd a real disk. Read-only `diskutil list/info` only (for fixtures).
The real stick test is the user's (checklist in `docs/testing-the-writer.md`).

- [x] PR 1 (ck-forge #91, merged) `refactor: put the disk operations behind a backend trait`: `DiskBackend` (list / prepare / raw device path) in ck-writer, Windows PowerShell as one impl, a fake for tests, no behaviour change on Windows.
- [x] PR 2 (ck-forge #92, merged; also fixes fsync ENOTTY on /dev/rdisk) `feat: write sticks on macOS`: diskutil impl (`list -plist`, `info -plist`, `unmountDisk`), only external removable USB physical disks, refuse boot/internal/synthesized/APFS/small/4K-sector, `/dev/rdiskN`; plist fixtures + refusal tests.
- [x] PR 3 (ck-forge #93, merged) `feat: macOS command line and CI`: `ck-forge stick write` with a clear sudo error, cfg gates in ck-admin-core / ck-base-image, macOS CI job, README + `docs/testing-the-writer.md` (macOS usage and manual test checklist).
- [x] PR 4 (ck-forge #94, green, fixed all review findings; blocked only by 1 outdated CodeRabbit thread that is not resolved: needs a human to resolve it, then `gh pr merge 94 --squash`) `feat: run the egui window on macOS`: app-egui builds and runs on macOS; stick-write screen shows CLI/sudo guidance.

## Notes
- Baseline: the workspace already builds and tests green on macOS (cargo test --workspace, 2026-11-24).
- macOS CI: hosted macOS runners did not start for this org (macos-14 is retired; macos-latest sat queued 15 min and was cancelled). The `macos-test` job is in pr.yml but runs only when repo variable `CK_MACOS_CI` is `true` (user task: set it once macOS runners work, or drop the job).
- Verified on this Mac without a real disk: `stick list` (read-only diskutil), and `tools/test-writer-disk-image.sh` (own hdiutil-attached scratch image: written, read back, bytes equal).
- Not done: QEMU firmware lookup for `image create` on macOS; Hub session in Keychain; Tauri window on macOS.
