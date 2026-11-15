# Cross-repo contracts

The source of truth for interfaces between components. SDKs are generated from these contracts; CI checks compatibility (openapi-diff). Changing a contract — PR here first, then implementations.

## Brain API (v0)

Headless, provider-agnostic (the brain never calls an LLM). OpenAPI: `contracts/brain.openapi.yaml` (generated from the `brain` code, vendored here on release).

- **Management plane** (`/admin/*`, project token): projects, actors, entities, facts, relations, proposals, audit.
- **Query plane** (`/context/*`, context token + `X-Brain-Actor`): context assembly. Response = `context` + mandatory `audit` (included / excluded+reason / substitutions).

## MCP tools (brain-mcp → brain API)

| Tool | Endpoint | Semantics |
|---|---|---|
| `get_context(query, intent?)` | POST `/context` | actor from the binding; response is an assembled block with citations |
| `get_portrait(name)` | GET `/entities?alias=` | entity card |
| `propose_memory(entity, text, source)` | POST facts (`status: pending`) | candidate; owner confirms |
| `search_audit(...)` | GET `/audit` | the log (for owners) |

**Tool description rule:** "call whenever the query mentions family members, gifts, health, plans" — model navigation lives in the description, not in core logic.

## Actor binding (channels)

- Telegram: `telegram_user_id` ↔ actor slug, bound during onboarding; child profile — PIN.
- Web (Next.js): account login = actor; child — PIN.
- The brain trusts only its own binding table; a channel cannot "assign" an actor that is not in the project.

## Portrait format

The external standard — the `portrait-spec` repo (MIT). The core stores facts in spec v0.2 form; format changes go through a change in `portrait-spec` with example migration.
