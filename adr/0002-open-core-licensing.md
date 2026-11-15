# ADR-0002: Licensing — three-layer open-core

Status: accepted (2026-11-15)

## Context
A solopreneur + open source: open source is a distribution and trust channel, but we also need protection from closed cloud competitors. Verified cases (Plausible, Postiz, Documenso, Bitwarden): free self-host + paid cloud works; MIT→AGPL relicensing happened *after* the threat appeared — so start in copyleft immediately.

## Decision
- Format spec — MIT (a standard for external developers).
- `project-specs` — CC-BY-4.0 (text).
- Core `brain`, `butler-bot`, `web-ui` — **AGPL-3.0** (the network clause blocks closed hosted forks).
- SDK, MCP, `self-host` — MIT (integrations and distribution).
- Cloud, add-on services, premium — proprietary.
- CLA from the first commit (the dual-license door stays open).

## Consequences
+ Self-host = marketing and a day-one right; a cloud competitor must open their changes.
− We cannot forbid "unmodified brain + someone's closed frontend" — closed by add-on services, trademark, and economics (the known limit of open-core).
