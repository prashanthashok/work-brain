# Work Brain

Single-user, local-first app for capture, delegation tracking, requirements, focus
and planning. Full design: `docs/design.md` (source of truth). Build state:
`docs/progress.md`. Slice prompts: `docs/prompts.md`.

## How to work in this repo

- Build ONLY the slice the user names. Do not generate later-phase endpoints,
  tables or screens. Phase 1 is split into slices 1a, 1b, 1c (see `docs/prompts.md`).
- Start each slice in plan mode. Show the plan, wait for approval, then build.
- If something conflicts with `docs/design.md`, stop and ask. Do not silently deviate.
- Finish a slice only when its exit criteria are demonstrated as observable behavior
  (see `docs/prompts.md`), then update `docs/progress.md` and commit.
- Never put secrets, real notes or personal data in the repo, tests, fixtures or logs.

## Stack (pin versions; no major upgrades by tag change)

- Backend: Spring Boot 3.x, Java 21, no preview features, port 9195.
- DB: PostgreSQL 18 in Docker, Flyway migrations (append-only, `infra/flyway/`,
  packaged under `db/migration`), UUID keys, `timestamptz`, date-only deadlines as `date`.
- Frontend: React 19, TypeScript, Vite, TanStack Query, port 6001 (strict port).
- Dependencies: Maven Central and public npm. Commit lockfiles. Never disable TLS verification.
- No Redis, broker, vector DB, separate search service or workflow engine.

## Layout

```text
backend/            Spring Boot app (domain packages = code boundaries)
frontend/           React app (feature folders under src/features)
infra/docker/       Compose files; paths must resolve to <root>/runtime and <root>/backups
infra/flyway/       Migrations
docs/               design.md, progress.md, prompts.md
runtime/postgres/   Live DB data (gitignored)
backups/            Backup archives (gitignored)
```

## Hard rules (from the design)

- Publish ports on 127.0.0.1 only. Browser uses relative `/api` URLs.
- Payloads are Java records, never JPA entities. UUID refs between aggregates,
  real FKs, no large bidirectional JPA graphs.
- Mutable aggregates: `id`, `created_at`, `updated_at`, optimistic `version`.
  Edits send expected version; stale = 409 and the editor keeps unsaved text.
- Retryable create/convert/accept use a client operation ID; same key + same input
  returns the original result, same key + different input returns 409.
- Errors: `ProblemDetail` + stable app error code via `GlobalExceptionHandler`.
  400 malformed, 404 missing, 409 state/version, 413 oversized. Never silently truncate.
- Literal text (brain dumps, code) is stored verbatim: no HTML stripping, rendered
  escaped in React. `@SanitizePlainText` only on fields defined as tag-stripped.
- Bound page sizes, allowlist filters/sorts, stable ordering with ID tie-breaker.
- No DB transaction held while waiting on AI or any external service.
- Copilot/AI is off by default and never blocks core flows. No fake working toggle.
- Backups: separate worker, `pg_dump` custom format, temp dir + checksum + atomic
  rename, exclusive lock, never copy a live PGDATA.
- Tests: PostgreSQL Testcontainers for DB behavior (not H2), Mockito at boundaries,
  `@JsonTest` for text fidelity. Frontend tests colocated.

## Commands

Fill in as they are created (build, test, `docker compose up`, lint). Keep this list current.
