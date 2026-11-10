# Templates

Files to copy into a repository when it gets its first code.

| File | Goes to | For |
|---|---|---|
| `rust/rustfmt.toml` | `rustfmt.toml` | Rust repos |
| `rust/deny.toml` | `deny.toml` | Rust repos |
| `rust/ci.yml` | `.github/workflows/ci.yml` | Rust repos |
| `web/ci.yml` | `.github/workflows/web.yml` | repos with a Svelte app |
| `release.yml` | `.github/workflows/release.yml` | every repo that ships releases |
| `release-please-config.json` | `release-please-config.json` | same; set `release-type` (`rust`, `node` or `simple`) |
| `.release-please-manifest.json` | `.release-please-manifest.json` | same |

Then add the `cargo` or `npm` ecosystem to the repository's `.github/dependabot.yml`.
