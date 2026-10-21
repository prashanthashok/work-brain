# Claude Code Prompts

Run one slice per session. Use plan mode, review the plan, then build. Start a fresh
session (`/clear`) per slice. State lives in the repo, `CLAUDE.md` and `docs/progress.md`.
After each slice: demonstrate the exit criteria, update `docs/progress.md`, commit.

## Slice 1a: Foundation

```
Read CLAUDE.md and docs/design.md (sections 2, 4, 5.12 storage layout, 8, 9).
Build slice 1a ONLY. Do not add capture, settings or backup features.

- backend/: Spring Boot 3.x, Java 21, port 9195, Flyway wired to infra/flyway/
  (packaged under db/migration) with an empty baseline migration, one health
  endpoint, ProblemDetail + GlobalExceptionHandler skeleton, local-profile config.
- frontend/: React 19 + TypeScript + Vite (port 6001, strictPort, /api proxied to
  9195), TanStack Query, app shell with header placeholder and a home screen that
  calls the health endpoint through a services/ HTTP layer using relative URLs.
- infra/docker/: compose with postgres:18 (pinned major; data mount to
  <root>/runtime/postgres via /var/lib/postgresql), app, and frontend static
  server/reverse proxy. Ports bound to 127.0.0.1 only: 6001->80 frontend,
  9195 app. No Postgres host port by default; add an explicit dev override file
  that exposes it on loopback.
- Compose paths must resolve to <root>/runtime and <root>/backups regardless of the
  working directory.
- README with run instructions. Lockfiles committed. No secrets in the repo; use
  .env.example and runtime-mounted secrets.

Exit criteria: `docker compose up` starts everything; http://127.0.0.1:6001 shows
the health result via the proxy; only one runtime/postgres directory exists;
backend and frontend tests run in CI script. Show me the plan first.
```

## Slice 1b: Settings and capture

```
Read CLAUDE.md and docs/design.md (sections 4, 5.1, 5.2, 6 "Capture without AI", 8).
Build slice 1b ONLY on top of 1a.

- Migrations: workspace_settings (time zone, global Copilot permission default
  false, permission revision, version) and capture_entry (original text, working
  text, optional title, state INBOX/PROCESSED/ARCHIVED, optional project ref as a
  nullable UUID column without an FK for now, timestamps, version), plus a
  table recording client operation IDs with request fingerprint.
- API: /api/settings (read, update with expected version), /api/captures
  (create with client operation ID, list with bounded pagination/allowlisted
  filters/stable ordering, get, edit working text with expected version,
  archive, reopen). Java records for payloads, ProblemDetail errors with stable
  codes, 400/404/409/413 behavior, bounded lengths, never truncate.
- Literal text preserved verbatim (code snippets, angle brackets, Unicode).
- UI: quick-capture panel reachable from every view, inbox list, capture detail/
  edit with explicit Save and pending/saved/failed state, 409 conflict keeps the
  unsaved text, settings page, header Copilot control shown as unavailable/off
  (no fake provider), loading/error/empty states.
- Tests: Postgres Testcontainers for migrations, constraints and concurrent
  edits; @JsonTest for text fidelity; idempotent replay and key-reuse 409;
  frontend tests for input preservation and conflict handling.

Exit criteria: save/list/edit/archive a literal capture in Docker; restart the
containers and the data is still there; duplicate submit with the same operation
ID creates one record. Show me the plan first.
```

## Slice 1c: Backup and restore

```
Read CLAUDE.md and docs/design.md (section 5.12 and 11 "Verify backup failures").
Build slice 1c ONLY on top of 1a and 1b.

- Independent backup worker container (not part of the app process) using the
  matching PostgreSQL client tools and a least-privilege backup role.
- Check backup age periodically; back up when >= 1 hour since the last successful
  backup, including catch-up after restart or sleep. Exclusive lock for scheduled
  and manual runs.
- pg_dump custom format into a temp directory with a manifest (completion time,
  Postgres major, schema/app compatibility, checksum), then atomic rename into
  <root>/backups. Never copy a live PGDATA. No credentials or cluster roles in
  the archive.
- Retention only after a successful backup (proposed 24 hourly / 7 daily / 4 weekly;
  make it configurable and confirm with me before changing the default).
- Read-only /api/operations/status from the worker's metadata showing last
  success, age, and failures, surviving app restarts.
- A documented restore script/procedure following design 5.12: verify checksum and
  compatibility, provision a fresh database and roles, restore with failure-on-
  error, reapply grants, start the app against it, inspect records. Never destroy
  the original source.
- Tests: a failed backup must not replace a valid archive; restore into an
  isolated fresh database in an automated test.

Exit criteria: hourly backup works; kill a backup mid-run and the last good archive
is intact; restore into a fresh database and read a saved capture back through the
app. Show me the plan first.
```

## After phase 1

Prompts for phases 2 to 6 should be written when you get there, one slice at a time,
using the corresponding design section and exit criterion from `docs/design.md`
section 10. Do not pre-write them; the design will move as you learn.
