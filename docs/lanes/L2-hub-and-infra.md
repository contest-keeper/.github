# L2: ck-hub and ck-infra

Read `README.md` in this folder first (rules, including rule 11: implement each item, open the PR, wait for green CI and squash-merge it yourself). You own **ck-hub** and **ck-infra** completely. Do not edit other repos.
Plan source: `sessions/plan-review-and-craft.md`. Deploys only through the repo pipelines: **never run `terraform apply` or an
`az` write**. Read-only `terraform validate/plan` and `az ... list/show` are fine when the user is signed in.
ck-hub tests need local PostgreSQL and `CK_TEST_DATABASE_URL`.

## Checklist (in order; steps 1 and 2 may be done in two worktrees one after the other)

### Step 1: ck-hub quick security and test fixes

- [x] **R3.2 silent pass.** (ck-hub#75) The store tests return early without `CK_TEST_DATABASE_URL` (`crates/*/tests/store.rs` near lines 21 and
  1818; `postgres_store.rs` near 2077). Skip with a visible reason; add `CK_TEST_DATABASE_REQUIRED=1` which fails when the URL is
  missing; set it in CI with a Postgres service container. If the shared workflow in the `.github` repo must change, add a
  line under L4 Requests (in L4's file) and set it in ck-hub's own workflow meanwhile.
- [x] **R2.13 CSRF.** (ck-hub#76, #77) List every state-changing route in `routes.rs` and check it rejects a non-JSON `Content-Type` and
  checks `Origin`. Fix gaps; add one integration test that sweeps all of them.
- [x] **R2.14 lockouts.** (docs route, ck-hub#78) Move login lockouts to PostgreSQL (migration, both store backends, tests) so `max_replicas` can rise;
  or, if not worth it now, write in `docs/` why it stays at one replica. No half-done state.

### Step 2: ck-infra hardening

- [x] **R2.11 Postgres firewall.** (ck-infra#44, deployed to develop) `infra/azure/main.tf` (about lines 72-78) has the `0.0.0.0` "Azure services" rule. Replace with VNet
  integration or a private endpoint if the student subscription allows it, otherwise the Container App's outbound IPs. Check what
  is allowed with read-only calls; record the choice in `docs/azure.md`; describe rollout order and rollback in the PR.
- [x] **R2.12 managed identity, Terraform half.** (ck-infra#45, #46; `moved {}` blocks kept, see PR) Managed identity for the Container App with least roles (Key Vault secrets read,
  storage access for user-delegation SAS); read `database-url` from Key Vault; fix the stale `bootstrap.sh` comment. Remove the
  Terraform `moved {}` blocks (about `main.tf:173-181`) only if the plan shows they were applied everywhere.
- [x] **R2.12 Hub half (waits for** (ck-hub#80 merged; link signing via identity is code-complete but NOT verified on real Azure and `CK_DOWNLOAD_KEY` is still set: removal waits for the user check in ck-hub docs/storage-identity.md) the Terraform half merged **and deployed by the pipeline):** switch downloads from
  `CK_DOWNLOAD_KEY` to a user-delegation SAS.
Done: `terraform fmt -check` and `validate` pass; PR shows the plan output or the reasoning if a plan is not possible locally;
`docs/azure.md` matches the resources.

### Step 3: Hub features still open and tests

- [x] **R2.10 Hub half:** (ck-hub#79) flag the same stick heartbeating from two places (design in a short note first, then build with tests).
- [x] Hub side of anything L1 or L3 lists under Requests below. (none listed at 2026-11-23)
- [x] **R3.3** (ck-hub#81) consolidate ck-hub's integration tests (16 binaries) into one `tests/it/main.rs` (move-only PR).
- [x] **R3.4** (ck-hub#82; ids.rs is in ck-protocol, L3's) judge and cut, one PR per crate, verdict lines in the PR (`ids.rs:108 tokens_compare_by_value`, `mail.rs:86`,
  `judge.rs the_submission_id_is_optional` are first candidates). Use the standard in `.github` `conventions/testing.md`
  (**waits for** L4 step 2; start with the keep rule: invariant, plausible breaking change, only this test).
- [x] **R3.5** (ck-hub#83) hostile-input sweeps for the Hub's heartbeat, event and enrolment handlers (proptest with shared `PROPTEST_CASES`);
  one test per security item above.

### Step 4: cleanup and review (R4.1 to R4.5 for Hub and infra)

- [x] R4.3 (ck-hub#96 ready and green, Forge develop verified on scoped routes + email; only the merge is pending, refused by the classifier; was held: ck-forge#74 moved develop to the org routes and email login, but the only released Forge build, 0.1.0-preview.72, still uses the unscoped routes and `username`; remove them from the Hub after a new Forge release is out) remove the unscoped "sole org" routes (`routes.rs:581`) and the `username` login field (`:1271`); `cargo machete`.
- [x] R4.4 (ck-hub#84, ck-infra#45) remove restating/history comments (`mail.rs:2`, `bootstrap.sh`); keep "why" and guard comments.
- [x] R4.5 (ck-hub#85 routes, #86 store, #87 postgres_store, #88 page; the Svelte fact is under L4 Requests) split `routes.rs` (2033 lines) per area, `store.rs` and `postgres_store.rs`, and `page.html` into separate JS and CSS
  files; fix the ck-hub `CLAUDE.md` "Svelte dashboard" claim only after verifying (L4 touches CLAUDE.md last: leave it to L4 and
  write the fact under L4 Requests).
- [x] R4.1 (notes written; ck-hub#89 auth sweep test) review sweep (auth, `enrol`, `signing`, `commands`, RLS first); findings in
  `sessions/review-2026-11-<dd>-ck-hub.md` and `...-ck-infra.md`, then small fix PRs.
- [x] #47 chrono (none in Cargo.lock, deny.toml or Cargo.toml): confirm none left in ck-hub (`Cargo.lock`, `deny.toml`).

## May touch

All of ck-hub and ck-infra.

## Needs the user

Organizer login on the dev Hub for: G4 acceptance (open the manifest and source-offer link), real Azure checks (Put Block,
Put Block List, Delete, read link, retention), the 100/300/500-stick capacity runs, the C7 cloud-cost approval, and the
`RELEASE_PLEASE_TOKEN`. Prepare the exact steps and scripts, then hand them to the user.

## Requests

- From L4 (2026-11-23): see `sessions/parallel/L4-findings.md`: nine Hub routes with no literal test mention (R3.5 input) and the live identity `id-ck-os-image-publish-dev` that Terraform does not define.

## Azure read-only checks (2026-11-24, az show/list and curl)

Develop Container App has no `CK_DOWNLOAD_KEY`; log says "storage links are signed with the managed identity" and both storage self-tests answer 404 (signature accepted); identity `id-ckhubdev-10lhs` has exactly Blob Delegator (account), Data Reader (downloads), Data Contributor (recordings), Key Vault Secrets User; `/download/forge` redirects to a signed link that answers 200; Postgres firewall has only the four `app-outbound-N` rules (no 0.0.0.0); max_replicas 1; blob soft delete is off, no lifecycle policy.
Still needs the user: Put Block / Put Block List / Delete / retention, G4, capacity runs, C7, RELEASE_PLEASE_TOKEN: exact steps in ck-hub PR #97 (`docs/user-checks.md`).
