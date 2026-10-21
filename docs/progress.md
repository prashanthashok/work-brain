# Progress

Update at the end of every slice. Keep it short; the repo and `docs/design.md` hold the detail.

| Slice | Scope | Status |
| --- | --- | --- |
| 0 | Repo, CLAUDE.md, design, prompts | Done |
| 1a | Docker Compose, Postgres 18, Flyway, Spring Boot health, React shell, proxy | Not started |
| 1b | Settings + capture (save/list/edit/archive), versioning, errors, idempotency | Not started |
| 1c | Backup worker, retention, fresh-database restore, `/api/operations/status` | Not started |
| 2 | Projects, people, work items, delegation, follow-ups, dependencies | Not started |
| 3 | Notes, search, stories, focus sessions, checkpoints | Not started |
| 4 | Availability, daily plans, scheduling | Not started |
| 5 | Copilot/AI add-on | Not started |
| 6 | External integrations (one at a time) | Not started |

## Decisions made

- Dependencies come from Maven Central and public npm (lockfiles committed).
- AI provider for phase 5 is undecided; core never depends on it.

## Open questions

- Initial text length limits (phase 1b).
- Backup retention policy (24 hourly / 7 daily / 4 weekly proposed) and a second-disk destination (phase 1c).
