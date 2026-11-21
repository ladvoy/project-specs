# ADR-0001: Storage — SQLite-first

Status: superseded by [ADR-0004](0004-storage-postgres-paradedb.md) (2026-11-21); was accepted 2026-11-15

## Context
One deployment = one project (a family). Requirements: "everything in one file" export, zero ops at the start, full-text search, later — semantics and multi-tenancy.

## Decision
SQLite (WAL) per project: JSON columns for cards/facts, FTS5 for search; the file is the backup/transfer unit. Semantic search in v0 — sqlite-vec or FTS+morphology. Postgres + JSONB + pgvector at the cloud stage; the schema migrates mechanically.

## Consequences
+ Zero ops; export = copying a file; self-host without DB administration.
− Concurrent writes by several actors are limited by WAL (fine for a family); Postgres migration is mandatory before the company preset.
