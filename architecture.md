# Architecture — the system skeleton

Approved gate for all component specs: components, request paths, identity, and the brain's output contract. Component specs (e.g. `brain/specs/001-brain-core`) must reference this skeleton and stay inside their component's boundary.

## Components and their boundaries

| Component | Role | Knows | Never does |
|---|---|---|---|
| **brain** | headless memory API: stores portrait cards; assembles **structured context blocks** per reader | project token + `X-Brain-Principal` header; card/fact data | natural-language answers; LLM calls; human identification; channel specifics (Telegram, ChatGPT, …) |
| **butler-bot** | messenger channel (Telegram first) | `tg_user_id ↔ principal` binding table (set at onboarding); child PIN | storing memory (stateless pass-through) |
| **brain-mcp** | MCP adapter for LLM chat clients (ChatGPT, Claude, DeepSeek, Hermes, Cursor…) | principal from its binding config; tool-description criteria | deciding *when* to answer — the client LLM decides using the tool description |
| **web-ui** | registration, wizards, portrait management, audit views | login = principal | bypassing the brain API |
| **client LLMs** (BYO) | produce the natural answer from the brain's context block | the context block + user query | seeing anything beyond the context block |
| ingestion / cron workers (later) | writes via the API with role `system` | — | — |

**One-line memory rule:** the brain is the single source of facts; channels, client LLMs, and profiles hold only manner and convenience, never facts.

## Request paths

**M — messenger:** person → Telegram → butler-bot resolves `tg_user_id → principal` → brain `/context` (`X-Brain-Principal` + project token) → context block → the channel's LLM (BYO, per-principal routing) → natural answer → person.

**C — LLM chat with MCP:** person talks to their ChatGPT / Claude / DeepSeek client → the client's MCP connection offers brain-mcp tools (`get_context`, `get_portrait`, `propose_memory`, `search_audit`) → the client LLM decides to call (criteria live in the tool description: family members, gifts, health, plans) → brain-mcp adds the configured principal → brain `/context` → context block returns as the tool result → the client LLM answers from it.

**A — admin:** owner → web-ui `/admin` → CRUD, confirm/reject proposals, export, deletion, audit views.

**Identity (weakest link of v0, stated honestly):** identity is established at the channel (binding table); the brain trusts the header and verifies registration; unknown principal → fail-closed (`everyone` facts only). Child: own account + PIN. Shared devices: charter + binding close ~90% of confusion; perfect anti-spoofing is not promised.

## The brain's output contract

The brain does **not answer** the user. Given a query and a principal, it returns a **structured context block**: matched subjects' facts with provenance, interpretation substitutions, meanwhile notes, and the audit of what was included, excluded, substituted, and why. The client LLM turns this block into the natural answer.

Quality bar (deterministic, therefore testable):

- **Completeness** — for the touched subjects, the block carries everything related: matched cards, cards connected via shared groups (v0's cross-card link — e.g. "gift for grandpa" surfaces the family gift-budget fact), core-marked facts of touched groups. No under-delivery.
- **Precision** — nothing unrelated: ranking (core first, then BM25, then recency) and the token budget cut the rest, with every cut visible in audit. No over-delivery.
- For a fixed (query, principal, project state) the expected include/exclude sets are fixed — this is what the golden suites will assert.

## Storage

PostgreSQL + ParadeDB pg_search from day one (ADR-0004, supersedes ADR-0001). Every row carries `project_id`; export/deletion is per-project (`COPY`/`DELETE`).

## Topologies

- **A — SaaS (v1):** our cloud (RU): brain + one butler per family + TG-bot; family members just message the bot.
- **B — self-host (later):** the family runs their own assistant runtime; the brain connects as their MCP server ("your data at home").

## Multi-tenancy

v0: one deployment = one project (token bound to it; `project_id` columns present from day one). Cloud: one container per family, inside it isolation via principal; orchestration/billing later.

## Repositories

| Repo | License | Contents |
|---|---|---|
| `portrait-spec` | MIT | The external portrait format standard (spec, JSON Schema, examples). Internal project knowledge does NOT live here |
| `project-specs` (this) | CC-BY-4.0 | The project's single source of truth |
| `brain` | AGPL-3.0 | Core: admin-api + context-api + ingestion worker + cron worker (role `system`) + migrations + golden tests + OpenAPI |
| `brain-mcp` | MIT | MCP adapter over `/context` for agents |
| `brain-sdk-ts`, `brain-sdk-python` | MIT | Thin API clients |
| `butler-bot` | AGPL-3.0 | Telegram butler: channels, principal↔TG binding, voice |
| `web-ui` | AGPL-3.0 | Next.js: registration, wizards, portrait management, audit views |
| `self-host` | MIT | docker-compose "up in 10 minutes" |
| `website`, `infra` | private | Landing pages, legal; our cloud configs/secrets/CI/CD |
