# Glossary — the shared language

Every agent and human reads this file before working on the project. A term missing from this glossary is a candidate for adding here, not for local invention.

## Entities and memory

- **Project** — one brain deployment: one family, one person, or one team. Types: `personal | family | smb | company` (role/policy presets).
- **Actor** — the human (or the `system` service role) on whose behalf a request is made. Determines access. Passed as the `X-Brain-Actor` header.
- **Entity / Portrait card** — a card: person, pet, place, organization, asset, project, topic, event. The `extensions` field holds anything outside the spec (Character Card V3 rule: unknown fields are preserved; own fields only inside extensions).
- **Fact** — the memory atom: one thought. Requires `source` (provenance). `valid_from/valid_to` — when it was true in the world (bi-temporal).
- **Domain** — a life area of a fact (food, health, hobbies, finances…). Users can add domains.
- **Polarity** — `like | dislike | neutral`.
- **Strictness** — `hard` (taboo, never in recommendations) / `soft` (avoid when alternatives exist) / `info` (plain knowledge). Only `dislike` carries strictness.
- **Audience** — fact access rights: `everyone | adults | owner | allow_list | role`. **Fail-closed**: no audience → visible to the owner only.
- **child_view** — an age-appropriate substitution of a fact for those who fail the gate. Dosing, not lying: the essence survives. No child_view → hidden from children entirely.
- **Constant core** — facts always in context (allergies, hard taboos). Limit ~500–1000 tokens per entity. Cores can differ per role (adult/child versions).
- **Keys** — keyword triggers on a fact (lorebook mechanics). Auxiliary to semantics and intent, never the only path.
- **Open questions** — "what we don't know yet"; fuel for the onboarding wizard.
- **Sharing: hold** — "a private note with delayed disclosure": the owner may allow disclosure later; to third parties — refusal. Reviewed monthly.
- **Proposal / pending** — a candidate fact from extraction or conversation; becomes a fact after the owner confirms.

## Mechanics

- **Assembly** — deterministic code behind `POST /context`: entity match → intent → audience filter → child_view substitution → constant core → ranking (relevance+importance+recency) → budget. **Isolation at input only.**
- **Audit** — the log of every assembly and every write: what was included, what was excluded and why, what was substituted. Assembly without audit does not conform.
- **Ingestion** — the write path: document upload → OCR → chunks → pending candidate facts.
- **Golden tests** — executable isolation scenarios on the reference example (the Carter family): "Rebecca must not see the prostatitis fact", "Rose must not see Becca's note". CI gate of the core.
- **Brain** — the headless memory API. **The brain never calls an LLM** — it assembles context; the client-side model does the thinking.
- **Butler** — the agent in the middle (Hermes profile / TG bot): voice and manners are its; knowledge and rights are the brain's.
- **Charter** — the family agreement on privacy and the single exception (safety).
- **Lad** — the informal short name of the product in speech ("add it to Lad").

## Don't confuse

- "the format spec" (portrait-spec, the external standard) ≠ "the project spec" (this repo) ≠ "a component behavior spec" (openspec in its own repo).
- `hard` dislike ≠ refusing a topic: a taboo affects **recommendations**, but a child's direct question about a health fact is served through `child_view`.
