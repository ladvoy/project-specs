# ADR-0004: Storage — PostgreSQL with ParadeDB pg_search from day one

Date: 2026-11-21
Status: accepted
Supersedes: [ADR-0001](0001-storage-sqlite-first.md) (storage-sqlite-first)

## Context

The portrait-card format is approved. The core (brain) goes straight to production: the target is a deployment serving ~1000 client projects without scaling panic. Data volume per project (a family) is small — hundreds of kilobytes — so raw engine speed is not the deciding factor.

Retrieval for this format is **lexical-first**: resolution by names and aliases, exact key triggers, facet filters (groups, time, visibility), and BM25 over fact texts. Semantic/vector search is not a day-one need.

The format needs a relational model (facts ↔ groups many-to-many, principals, roles), transactional writes with mandatory audit, and full per-project export/deletion (data rule D7). The self-host bundle ("running in 10 minutes") remains a requirement.

## Decision

1. **PostgreSQL 17+ is the only system of record**, from day one — no embedded-DB staging step. Every row carries `project_id`; one instance serves thousands of tiny projects.
2. **ParadeDB `pg_search` is the search layer**: BM25 scoring (Tantivy-based), facets, aggregations — an extension inside the same Postgres, ACID, no second service to deploy or sync. The self-host compose ships the ParadeDB image.
3. **Lexical-first retrieval.** Exact match: `keys`/`aliases` as `text[]` with GIN indexes; fuzzy alias resolution: `pg_trgm`. BM25 over fact texts via `pg_search`. Native `tsvector` + `ts_rank_cd` is the fallback where `pg_search` cannot run (minimal self-hosts).
4. **Vectors are deferred.** Semantic search (paraphrase, cross-language queries) is added later as a `pgvector` column on facts when a proven need appears — no engine change required.
5. **Visibility is enforced in application code** (fail-closed, spec §2.3). The database stores visibility data but never decides access.
6. **Export and deletion (D7):** per-project `COPY … TO` and `DELETE … WHERE project_id = ?` — nothing lives outside the project's rows.
7. **First-cut schema:** `projects`; `principals(slug, attributes jsonb, card)`; `roles(project, name, members text[])`; `cards(slug, type, name, aliases text[], defaults jsonb)`; `facts(id, card, text, author, status, source jsonb, time_from, time_to, revalidate, groups text[], visibility jsonb, core text[], keys text[], interpretation jsonb, disclosure jsonb, extensions jsonb)`; `groups`; `fact_groups` (m2m); `audit` (append-only).

## Alternatives considered

- **SQLite-first (ADR-0001)** — simplest self-host (one file, zero DBA), and the reason it was chosen on 2026-11-15. Superseded: the product target moved to production-first with many concurrent clients; Postgres was the planned migration anyway, and migrating later under load is the "panic scaling" this ADR avoids.
- **Redis 8 / Valkey 8.1 (+ search modules)** — fast and in-RAM (Valkey-search GA, BSD; Redis Query Engine in core, AGPLv3 option), but persistence, export, and audit as a *system of record* are worse; at our volumes the speed win is ≈ 0. Rejected as truth storage; acceptable later as cache/pubsub in the cloud deployment.
- **LanceDB / Chroma / DuckDB** — embedded or analytics engines: single-process, no multi-client transactional story; DuckDB is columnar OLAP. The format is relational with audit — Postgres' home turf. Rejected as system of record; LanceDB is reconsiderable later as a specialized vector index, if ever needed.
- **Meilisearch / Typesense** — excellent instant-search accelerators, but a second service and not a system of record. Rejected for now.

## Consequences

- The self-host bundle grows a Postgres container (ParadeDB image) — heavier than SQLite's zero-ops file, accepted for production readiness.
- Golden tests and the brain MVP run against the same Postgres schema; local development uses the ParadeDB docker image.
- Multilingual BM25 (Russian/English stems) is handled by `pg_search` analyzers when non-English projects arrive.
- No migration path is needed from the SQLite sketches: nothing was implemented.
