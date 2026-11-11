# Templates

Files to copy into a repository when it gets its first code. Every repository already has
`.editorconfig`, `.gitattributes`, `.gitignore`, `CODEOWNERS`, `CLAUDE.md`, Dependabot and the PR checks.

## Rust repositories

| File | Goes to |
|---|---|
| `rust/Cargo.toml` | `Cargo.toml` (workspace: shared lints and package fields; replace `REPO`) |
| `rust/rust-toolchain.toml` | `rust-toolchain.toml` |
| `rust/rustfmt.toml`, `rust/clippy.toml`, `rust/deny.toml` | repository root |
| `rust/ci.yml` | `.github/workflows/ci.yml` |
| `vscode/rust/*` | `.vscode/` |

## Web projects (TypeScript + Svelte)

| File | Goes to |
|---|---|
| `web/package-scripts.json` | merge into `package.json` (scripts, `packageManager`, dev tools) |
| `web/.prettierrc.json`, `web/eslint.config.js`, `web/tsconfig.json` | project root |
| `web/.nvmrc` | project root |
| `web/ci.yml` | `.github/workflows/web.yml` |
| `vscode/web/*` | `.vscode/` |

## Releases

| File | Goes to |
|---|---|
| `release.yml` | `.github/workflows/release.yml` |
| `release-please-config.json` | root; set `release-type` (`rust`, `node` or `simple`) |
| `.release-please-manifest.json` | root |

After copying, add the `cargo` or `npm` ecosystem to `.github/dependabot.yml`, and add the new CI
check to the `main` ruleset.
