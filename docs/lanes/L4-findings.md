# L4 findings: flows artifact cross-check (step 3), 2026-11-23

## Status

- **Artifact not updated.** `https://claude.ai/artifact/TE44nJbEz8dJsBzfSkxYKb` cannot be read from this session
  ("artifact not found / not shared with you"; it is not in the account's artifact list either). Without its text I could not
  compare its claims, and I did not publish a new artifact. **Needs the user:** open/share it with the account this session
  uses, or paste its text; then the tab-by-tab comparison below can be finished in minutes. The facts below are what the
  artifact should say.
- **az:** signed in (`az account show` works). The default subscription is the employer's (`sub-assu-dev`), which holds nothing of
  Contest Keeper. The Contest Keeper groups are in **"Azure for Students"** (a different account); I used read-only
  `az resource list -g <group> --subscription "Azure for Students"` only and changed no default or setting.

## Azure: live vs `ck-infra` on `origin/develop` (L2 is changing it)

| Group | Live resources | In Terraform? |
|---|---|---|
| `rg-ckhub-dev` | PostgreSQL flexible server `psql-ckhubdev-10lhs`, Key Vault `kv-ckhubdev-10lhs`, Container App `ca-ckhubdev-10lhs` (francecentral), storage `ckhubdevdl10lhs` (downloads, recordings), email service + domain `mail.dev.contestkeeper.com`, communication service `cs-ckhubdev-10lhs`, storage `ckwebdevlt84z` (website, `infra/website`) | all yes (`main.tf`, `mail.tf`, `infra/website/main.tf`) |
| `rg-ckhub-dev` | user-assigned identity **`id-ck-os-image-publish-dev`** | **No.** Nothing in `infra/` defines it; created by hand or by another repo's workflow. Drift to record (ck-os image publishing). |
| `rg-ckhub` (production) | empty | Group made by `bootstrap.sh`; production was never applied. |
| `rg-ckhub-tfstate` | storage `ckhubtfexgsdbwqoy` (state) | `bootstrap.sh` |
| `rg-kc-dash` | Container Apps environment `kc-dash-env` (+ managed certificate for the Hub), job `kc-dash-run`, log workspace `kc-dash-logs` | not ours: the develop Hub joins `kc-dash-env` through `shared_environment_*` in `develop.tfvars`; the Terraform `moved {}` blocks (`main.tf:173-181`) are still there (R4.3) |

Develop Hub has no own Log Analytics or environment (as `develop.tfvars` says: it uses the shared one). The Entra applications and
OIDC credentials made by `bootstrap.sh` are not resources and were not listed (needs `az ad`, not run).

Still true in `main.tf` on `develop` (R2.11/R2.12): the `0.0.0.0` "azure_services" firewall rule (`azurerm_postgresql_flexible_server_firewall_rule.azure_services`).

## Hub routes (`ck-hub/crates/ck-hub/src/routes.rs`, `develop`)

57 distinct paths. Every one is listed below with the test files whose source contains the path (regex match, so a path built
with `format!` and a helper can still show "NONE": verify by hand). Tests per file: api 32, accounts 12, commands 9, enrol 10,
forge_connect 8, invitation_routes 4, profiles 9, recordings 12, signing 7, signup 14, sticks 16, startup 2, store 11.

| Route | Test files that mention it |
|---|---|
| `/` | api  |
| `/api/v1/config-hashes` | sticks  |
| `/api/v1/config-signatures` | **NONE** |
| `/api/v1/config-signing-key` | signing  |
| `/api/v1/enrol` | enrol  |
| `/api/v1/enrol/claim` | enrol  |
| `/api/v1/enrolment-codes` | **NONE** |
| `/api/v1/forge/connect` | forge_connect  |
| `/api/v1/forge/token` | forge_connect  |
| `/api/v1/hardware-writes` | **NONE** |
| `/api/v1/images` | api  |
| `/api/v1/invitations/accept` | api invitation_routes  |
| `/api/v1/login` | accounts api commands enrol forge_connect profiles recordings signing signup sticks store  |
| `/api/v1/logout` | accounts  |
| `/api/v1/me` | accounts invitation_routes  |
| `/api/v1/orgs/{slug}/commands/{command_id}` | commands  |
| `/api/v1/orgs/{slug}/config-hashes` | **NONE** |
| `/api/v1/orgs/{slug}/config-signatures` | signing  |
| `/api/v1/orgs/{slug}/enrolment-codes` | enrol  |
| `/api/v1/orgs/{slug}/hardware-writes` | enrol  |
| `/api/v1/orgs/{slug}/invitations` | invitation_routes  |
| `/api/v1/orgs/{slug}/invitations/{id}` | invitation_routes  |
| `/api/v1/orgs/{slug}/members` | invitation_routes  |
| `/api/v1/orgs/{slug}/members/{user_id}` | invitation_routes  |
| `/api/v1/orgs/{slug}/recordings/{recording_id}/view` | recordings  |
| `/api/v1/orgs/{slug}/sticks` | accounts commands enrol recordings signing sticks  |
| `/api/v1/orgs/{slug}/sticks/{id}` | commands enrol recordings signing sticks  |
| `/api/v1/orgs/{slug}/sticks/{id}/archive` | recordings sticks  |
| `/api/v1/orgs/{slug}/sticks/{id}/commands` | commands  |
| `/api/v1/orgs/{slug}/sticks/{id}/credential` | enrol sticks  |
| `/api/v1/orgs/{slug}/sticks/{id}/network` | **NONE** |
| `/api/v1/orgs/{slug}/sticks/{id}/recordings` | recordings  |
| `/api/v1/orgs/{slug}/sticks/{id}/restore` | **NONE** |
| `/api/v1/orgs/{slug}/sticks/{id}/submissions` | **NONE** |
| `/api/v1/password-reset` | api signup  |
| `/api/v1/password-reset/confirm` | api signup  |
| `/api/v1/profiles` | profiles  |
| `/api/v1/profiles/{id}` | profiles  |
| `/api/v1/profiles/{id}/wallpaper` | profiles  |
| `/api/v1/recordings` | recordings  |
| `/api/v1/recordings/{id}` | recordings  |
| `/api/v1/recordings/{id}/chunks/{index}` | recordings  |
| `/api/v1/recordings/{id}/complete` | recordings  |
| `/api/v1/register` | api signup  |
| `/api/v1/staff/organizations` | api invitation_routes  |
| `/api/v1/sticks` | accounts api forge_connect sticks store  |
| `/api/v1/sticks/{id}/commands` | **NONE** |
| `/api/v1/sticks/{id}/credential` | sticks  |
| `/api/v1/sticks/{id}/network` | api  |
| `/api/v1/sticks/{id}/recordings` | **NONE** |
| `/api/v1/sticks/{id}/submissions` | api  |
| `/api/v1/verify-email` | api signup  |
| `/download/forge` | api  |
| `/download/image` | api  |
| `/download/image/{id}` | api  |
| `/download/image/{id}/source-offer` | api  |
| `/forge/connect` | forge_connect  |

Paths with **no literal mention in any test** (candidates for R3.5, confirm first): `/api/v1/config-signatures`, `/api/v1/enrolment-codes`,
`/api/v1/hardware-writes`, `/api/v1/orgs/{slug}/config-hashes`, `/api/v1/orgs/{slug}/sticks/{id}/network`, `.../restore`, `.../submissions`,
`/api/v1/sticks/{id}/commands`, `/api/v1/sticks/{id}/recordings`. Several are probably reached through helpers in `signing.rs`, `enrol.rs`
and `commands.rs`; a reader must confirm, which is why they are findings and not gaps yet.

## Signed-settings flow (to add to the artifact)

Protocol v0.5.0 (ck-protocol #40: signed stick settings) -> Hub signs: `GET /api/v1/config-signing-key`, `POST /api/v1/config-signatures`
and `/api/v1/orgs/{slug}/config-signatures` (ck-hub #74, tests in `tests/signing.rs`, 7) -> Forge asks for the signature when it writes a stick
(ck-forge #68) -> the stick agent only accepts settings signed by the Hub (ck-os #79, `ck-agent-config`) -> the Hub signing key is passed in by
Terraform (ck-infra #43). Open: the image workflow publishes without `CK_CONFIG_PUBLIC_KEYS` (R2.1), the ESP fallback remains.

## Not done in this step

- Claim-by-claim check of the artifact (blocked above), including the coverage-matrix labels (unit, integration, qemu-smoke CK#, ck-vmtest, manual).
  The qemu-smoke CK ids and ck-vmtest were not mapped here.
- `az ad` (Entra apps) and any Azure write: not run.

## Hand-over to L1 to L3 (R3.5 input)

- **L2 (Hub/infra):** the nine route paths above; `id-ck-os-image-publish-dev` is missing from Terraform (add or document); production group is empty.

# Step 4: ck-website licences panel vs `ck-os/docs/licences.md` and the source offer (2026-11-23)

Compared `ck-website` `site/src/App.svelte` (the Licences panel, `origin/develop`) with `ck-os` `docs/licences.md` and `image/SOURCE-OFFER.txt.in`
(`origin/develop`). The panel is accurate where it speaks; it is shorter than the docs. No website change is needed now. Drift, most important first:

1. **`licences.md` is stale on the source offer.** Its "Source offer (to do)" section says the file is still to be added, but it is built and
   shipped (`tools/build-image` writes `SOURCE-OFFER.txt` next to the image and into `/usr/share/doc/keeper-os/`, `tools/publish-image` uploads it,
   smoke check CK25 covers it). Owner: L1 (ck-os docs). The panel's claim "a written offer of its source code, next to the download on your Hub"
   is true (`GET /download/image/{id}/source-offer` exists in the Hub), but no Hub test mentions that path literally (see step 3 findings).
2. **Firefox:** the panel and the source offer both list Firefox (MPL 2.0, Mozilla apt repository); `licences.md` still says "Not checked yet" and
   "read the Mozilla trademark policy". The panel names a product whose trademark policy the doc says is unread. Owner: L1/user to decide (decision 14 area).
3. **Debian profile:** `image/profiles/debian/SOURCE-OFFER.txt.in` exists, so a second family is built; `licences.md` and the panel only talk
   about Ubuntu packages and the Canonical trademark. A Debian image needs the same trademark check (Debian logo/name policy), not recorded anywhere.
4. **Components named in the offer but not on the panel:** Kotlin compiler (Apache 2.0) and the clangd extension (MIT); the doc's OpenJDK, Python, GCC row is
   also not named on the panel ("separate programs ... mostly GPL" covers it loosely). Acceptable; add the names only if the user wants a full list.
5. **Canonical trademark sentence:** the offer says "not made, approved or endorsed by Canonical"; the panel has no such line. Optional to mirror it.
6. **Status:** `licences.md` is still headed "draft for the owners to approve" (licence approval #31 and the Canonical question are the user's, README "Needs the user").
7. The panel's Apache-2.0 statement matches `licences.md`; the Apache/permissive dependency claim rests on `deny.toml` in each repo (not re-checked here).

Nothing else is open for the website; no other issues found in the panel.

# Re-check 2026-11-24 (from the repos on this Mac; read-only az)

- **Azure vs `ck-infra` develop:** `rg-ckhub-dev` = Terraform-defined resources plus the hub identity `id-ckhubdev-10lhs` (in `main.tf`, the earlier table missed it) and the hand-made `id-ck-os-image-publish-dev` (in no repo's Terraform: drift; ck-os `tools/publish-image` does not create it). `rg-ckhub` still empty, `rg-ckhub-tfstate` one storage account, `rg-kc-dash` unchanged (shared env, not ours). Subscription "Azure for Students" has all four groups; subscription `contest-keeper` lists no resources.
- **Hub routes:** 63 distinct route paths in `ck-hub/crates/ck-hub/src/routes/*.rs` (now a directory); every one is mentioned by at least one test in `crates/ck-hub/tests/it/` (loose text match). The seven earlier "NONE" routes (config-signatures, enrolment-codes, hardware-writes, org config-hashes, sticks network/restore/submissions) are now covered. Signed-settings flow: tests in `tests/it/signing.rs`.
- **House rules:** caller workflow merged in all six repos (.github#32-#34 fixed the marker-extraction bug).
- **Audit:** open PRs: ck-protocol#50 (release 0.5.1), ck-forge#89 (docs). Local unmerged branches remain in ck-forge (15), ck-hub (feature/forge-sso, feature/org-profiles), ck-protocol, ck-os: classify/delete only with the user. Disk fine (465 GiB free).
- **Still needs the user / Windows:** the flows artifact text (private, unreadable), `sessions/` and `IMPLEMENTATION-PLAN.md` (to tick boxes and update lane status), `D:\ck-wt`, `D:\ck-images`, stray folders in `D:\contest-keeper`, Entra apps (`az ad`).
