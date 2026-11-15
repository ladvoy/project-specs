# Architecture: repo map and data flows

## Repositories

| Repo | License | Contents |
|---|---|---|
| `portrait-spec` | MIT | The external portrait format standard (spec, JSON Schema, examples). Internal project knowledge does NOT live here |
| `project-specs` (this) | CC-BY-4.0 | The project's single source of truth |
| `brain` | AGPL-3.0 | Core: admin-api + context-api + ingestion worker + cron worker (role `system`) + migrations + golden tests + OpenAPI |
| `brain-mcp` | MIT | MCP adapter over `/context` for agents |
| `brain-sdk-ts`, `brain-sdk-python` | MIT | Thin API clients |
| `butler-bot` | AGPL-3.0 | Telegram butler: channels, actor↔TG binding, voice |
| `web-ui` | AGPL-3.0 | Next.js: registration, wizards, portrait management, audit views |
| `self-host` | MIT | docker-compose "up in 10 minutes" |
| `website`, `infra` | private | Landing pages, legal; our cloud configs/secrets/CI/CD |

## Data flows

```
[family members] → TG → butler-bot ─┐
[family] → web-ui ──────────────────┼→ brain /context (X-Brain-Actor) → audit
[Claude/Cursor] → brain-mcp ────────┘         ↑
[Hermes] → brain-sdk-python ──────────────────┘
[owner] → web-ui /admin → brain: CRUD + ✓/✗ proposals
[cron worker] → brain: role=system (hold review, pending reminders)
[PDF upload] → brain: ingestion → pending facts
```

**The project DB is the only source of truth; all writers go through the API → everything is audited.** Channels and UI are memory-stateless.

## Storage

v0: SQLite per project (the file = export/backup) + FTS5; later Postgres + JSONB + pgvector (mechanical migration). See ADR-0001.

## Multi-tenancy

v0: one deployment = one project. Cloud orchestration — one container/OS user per family (Hermes SECURITY.md §2.2 guide), later.
