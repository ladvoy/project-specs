# Архитектура: карта репозиториев и потоки данных

## Репозитории

| Репо | Лицензия | Содержимое |
|---|---|---|
| `portrait-spec` | MIT | Внешний стандарт формата портретов (спека, JSON Schema, примеры). Внутренние знания проекта — не здесь |
| `project-specs` (этот) | CC-BY-4.0 | Центральная правда проекта |
| `brain` | AGPL-3.0 | Ядро: admin-api + context-api + ingestion worker + cron worker (роль `system`) + миграции + golden-тесты + OpenAPI |
| `brain-mcp` | MIT | MCP-адаптер над `/context` для агентов |
| `brain-sdk-ts`, `brain-sdk-python` | MIT | Тонкие клиенты API |
| `butler-bot` | AGPL-3.0 | Telegram-дво́рцкий: каналы, привязка actor↔TG, голос |
| `web-ui` | AGPL-3.0 | Next.js: регистрация, визарды, управление портретами, audit |
| `self-host` | MIT | docker-compose «за 10 минут» |
| `website`, `infra` | приватные | Лендинги, оферта; конфиги/секреты/CI/CD нашего облака |

## Потоки данных

```
[члены семьи] → TG → butler-bot ─┐
[семья] → web-ui ────────────────┼→ brain /context (X-Brain-Actor) → audit
[Claude/Cursor] → brain-mcp ─────┘         ↑
[Hermes] → brain-sdk-python ───────────────┘
[владелец] → web-ui /admin → brain: CRUD + ✓/✗ proposals
[cron worker] → brain: role=system (hold-ревизия, pending-напоминания)
[загрузка PDF] → brain: ingestion → pending-факты
```

**Единственный владелец правды — БД проекта; все writers проходят через API → всё в audit.** Каналы и UI stateless по памяти.

## Хранение

v0: SQLite per project (файл = экспорт/бэкап) + FTS5; позже Postgres + JSONB + pgvector (миграция механическая). См. ADR-0001.

## Мультитенантность

v0: один деплой = один проект. Облачная оркестрация — контейнер/OS-юзер на семью (гайд Hermes SECURITY.md §2.2), позже.
