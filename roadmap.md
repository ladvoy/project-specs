# Roadmap

## v0 — "the butler answers" (core, solo dogfood)
- [ ] `brain`: FastAPI skeleton — admin-api + context-api, SQLite, OpenAPI from code
- [ ] Golden isolation tests on the Carter reference (CI gate)
- [ ] Ingestion v0: files → text → chunks → pending facts
- [ ] `brain-mcp` + connecting to Hermes (dogfood: Scenario 3 in two roles)
- [ ] `project-specs` + `portrait-spec` published (this repo is already here)

## v1 — "the family uses it" (closed beta)
- [ ] `butler-bot`: TG channel, actor binding, child_view assembly
- [ ] Onboarding questionnaire (open_questions) + propose_memory with confirmation
- [ ] `web-ui` (Next.js): registration, wizards, portraits, audit views
- [ ] `self-host`: compose "up in 10 minutes"
- [ ] Semantic search + ranking (relevance/importance/recency)

## v2 — "beyond the family"
- [ ] Cloud: one container per family, billing, domain
- [ ] Add-on services 🔧: web search, local events, HR bot
- [ ] company preset: roles, sections, SSO
- [ ] Localizations: folklore brand wrappers per market (Dovlat, "home angel"…)

Ordering and scope — see the internal archive; details are refined through the SDD pipeline (Spec Kit when a component is born, OpenSpec in the steady flow).
