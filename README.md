# project-specs

The single source of truth for the **Ladvoy** project: vision, shared language, invariants, cross-repo contracts, architecture, and decisions.

This is the L1 knowledge layer: every agent session and every repository of the project vendors its shared knowledge from here. **Component behavior specs do NOT live here** — they live in their own repos (`openspec/specs/`) and travel as deltas in the same PRs as code.

## Layout

```
product/        — vision, personas, use-case index
glossary.md     — the shared language (every agent and human reads this first)
invariants.md   — the constitution: immutable project rules
contracts/      — cross-repo contracts (OpenAPI, MCP tools, actor binding)
architecture.md — repo map and data flows
adr/            — architecture decision records
roadmap.md      — milestones
```

## The boundary rule

This repo holds only what **does not belong to a single component**: language, invariants, cross-repo contracts, architecture, decisions. Anything describing a component's behavior lives next to that component's code.

## How to use

- A new agent session in any repo: reads the local `AGENTS.md` → which points to vendored sections from here (constitution, glossary).
- Vendoring: a copy of the section in the repo, with a comment `source: project-specs@<commit>`. When this repo changes — run the "update vendors" checklist in all repos.
- `contracts/` is the source for SDK generation and CI compatibility checks (openapi-diff).

## License

[CC-BY-4.0](./LICENSE) — documentation and standards must be freely shareable with attribution.

Ladvoy — "the one who keeps the home in harmony."

© 2026 Boris Bogdanovich
