# Shell, systemd and the image

Used by Keeper OS (image patch, services on the stick) and the Hub setup.

## Shell scripts

- Prefer Rust for anything with logic. Shell is for short glue (under ~100 lines).
- Start with `#!/usr/bin/env bash` and `set -euo pipefail`. Quote every variable: `"$var"`.
- Must pass `shellcheck` and be formatted with `shfmt -i 2 -ci`.
- File names `kebab-case.sh`; functions `snake_case`; constants `UPPER_CASE`.
- Print errors to stderr and exit non-zero. Clean up temp files with `trap`.
- Scripts that change an image check their result (e.g. `e2fsck -fn`) before finishing.

## systemd units

- Our units are named `ck-<thing>.service` / `.timer`; drop-ins for other units go in
  `<unit>.d/10-ck.conf`.
- Every service sets `Description=`, explicit `After=`/`Wants=`, `Restart=on-failure` where it makes sense.
- Harden by default: `NoNewPrivileges=yes`, `ProtectSystem=strict`, `ProtectHome=yes`, `PrivateTmp=yes`,
  `ReadWritePaths=` only what is needed; run as a dedicated user unless root is required.
- Nothing may slow down the boot: no waiting on the network before the desktop, no `Type=oneshot` that
  can hang without `TimeoutStartSec=`.

## Paths on the stick

- Config: `/etc/ck/`. State: `/var/lib/ck/`. Logs: the journal (no own log files).
- Per-stick settings: `ck/config.json` on the EFI partition, readable by root only.
- Everything we add to the image is listed in the image patch; nothing is changed by hand.

## Docker (Hub setup)

- Images pinned by version (never `latest`), non-root user, health checks on every service.
- Configuration through environment variables; secrets through files or the platform's secret store, never in Git.
