# Database

PostgreSQL, accessed with `sqlx` (checked queries, no ORM).

## Naming

- Tables: plural `snake_case` (`sticks`, `submission_matches`). Columns: `snake_case`.
- Primary key `id` (UUIDv7). Foreign keys `<table_singular>_id` (`stick_id`).
- Every table has `created_at timestamptz not null default now()`; mutable tables also `updated_at`.
- Indexes: `ix_<table>_<columns>`; unique: `ux_<table>_<columns>`; foreign keys: `fk_<table>_<column>`.
- Always `timestamptz` (UTC), never `timestamp`. Money and counts as integers. Enums as `text` with a `check` constraint.

## Migrations

- `sqlx migrate` files in `migrations/`, named `<yyyymmddhhmmss>_<what>.sql`.
- Migrations only move forward. Never edit one that has been merged; add a new one.
- Changes must work while the old version still runs: add a column → deploy code → drop the old column
  in a later release.
- Big tables: create indexes `concurrently`; backfill in batches.

## Queries

- Use `sqlx::query!` / `query_as!` so queries are checked at build time (`.sqlx/` offline data is committed).
- Always parameters, never string formatting.
- Many writes that belong together go in one transaction.
- Store event ids with a unique constraint so a resent event is ignored, not duplicated.

## Data

- Keep only what is needed. Personal data (usernames, IPs) has a retention period set per contest.
- No passwords, cookies or session ids, ever. Organizer passwords only as Argon2id hashes.
