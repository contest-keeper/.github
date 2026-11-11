# API and protocol

Covers the Hub's HTTP API and the events sticks send to it.

## HTTP

- Base path `/api/v1/`. A breaking change means `/api/v2/`, served next to v1 until clients move.
- Resources are plural nouns in kebab-case: `/api/v1/sticks`, `/api/v1/contests/{id}/submissions`.
- JSON bodies with `snake_case` fields. Timestamps are RFC 3339 UTC strings (`2026-11-10T09:30:00Z`).
  Ids are strings (UUIDv7 for new records).
- Methods: `GET` reads, `POST` creates or runs an action, `PATCH` changes, `DELETE` removes. `GET` never changes state.
- Status codes: `200`, `201` (+ `Location`), `204`, `400` bad input, `401` not signed in,
  `403` not allowed, `404`, `409` conflict, `422` valid JSON but rejected, `429`, `500`.
- Errors use one shape everywhere (RFC 9457 problem details):
  `{"type": "...", "title": "Stick not registered", "status": 404, "detail": "...", "code": "stick_not_found"}`.
  `code` is stable and meant for programs; `title`/`detail` are for people.
- Lists are paginated with a cursor: `?limit=50&cursor=...` → `{"items": [...], "next_cursor": "..."}`.
- The OpenAPI spec is generated from the code and checked in CI; clients generate their types from it.
- Every request gets a request id (`X-Request-Id`), logged on both sides.

## Stick events

- Event types live only in `ck-protocol`. Field names `snake_case`, type names `PascalCase` in Rust.
- Every event has: `event_id` (UUIDv7, used to drop duplicates), `stick_id`, `sent_at`, `seq`
  (counter per boot), `kind`, and the payload.
- Events are signed (`X-CK-Signature` over the exact body bytes) and are safe to resend:
  the Hub accepts the same `event_id` twice and stores it once.
- Never send passwords, cookies, session ids or full source code. Send fingerprints (SHA-256) instead.

## Compatibility

- Adding an optional field is fine. Removing or renaming a field, or changing its meaning, is breaking.
- Readers ignore unknown fields. Writers never send `null` for "missing"; they leave the field out.
- `ck-protocol` follows SemVer; a breaking change is a major version and needs a decision record.
- The Hub accepts the current and the previous major protocol version, so sticks made for one
  contest keep working while the Hub is upgraded.
