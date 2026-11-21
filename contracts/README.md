# Cross-repo contracts

The source of truth for interfaces between components. SDKs are generated from these contracts; CI checks compatibility (openapi-diff). Changing a contract — PR here first, then implementations.

## Brain API (v0)

Headless, provider-agnostic (the brain never calls an LLM).

OpenAPI: [`brain.openapi.yaml`](./brain.openapi.yaml)  
`source: ladvoy/brain` — `specs/001-brain-core/contracts/openapi.yaml` (includes ingest jobs from `002-ingestion`; served live at `/openapi.json`).

Base path: `/v1`.

| Plane | Auth | Paths | Consumers |
|---|---|---|---|
| **Privileged** | Bearer project (admin) token | `/projects/{id}/…` admin CRUD, import/export, proposals, audit | web-ui admin, internal bots (`system`) |
| **Query** | Bearer context token + `X-Brain-Principal` | `POST /projects/{id}/context` | butler-bot, brain-mcp, user views |

Response of `/context` = structured context block (`facts`, `notes`, `audit`) — never a natural-language answer. Audit lists included / excluded+reason / substitutions.

Re-vendor checklist: when brain's OpenAPI changes, copy into this file, bump the `source:` commit above, open a PR here before dependent SDKs.

## MCP tools (brain-mcp → brain API)

| Tool | Endpoint | Semantics |
|---|---|---|
| `get_context(query, intent?)` | `POST /v1/projects/{id}/context` | principal from the channel binding; assembled block + audit |
| `get_portrait(slug)` | `GET /v1/projects/{id}/cards/{slug}` | card with facts (admin token) or future read path |
| `propose_memory(card, text, source?)` | `POST …/cards/{slug}/facts` | `status: pending` when source missing; owner confirms |
| `search_audit(...)` | `GET /v1/projects/{id}/audit` | append-only log |

**Tool description rule:** "call whenever the query mentions family members, gifts, health, plans" — model navigation lives in the description, not in core logic.

## Principal binding (channels)

Channels resolve a human to a **principal slug** and send it as `X-Brain-Principal`. The brain does not identify people.

- Telegram: `telegram_user_id` ↔ principal slug (onboarding); child profile — PIN.
- Web: account login = principal; child — PIN.
- The brain trusts only its own principals table; a channel cannot invent a principal that is not in the project. Unknown slug → fail-closed everyone-only assembly (FR-002).

## Portrait format

The external standard — the [`portrait-spec`](https://github.com/ladvoy/portrait-spec) repo (MIT). The brain imports/exports conforming project JSON. Format changes go through `portrait-spec` (semver + example migration); code follows the spec.
