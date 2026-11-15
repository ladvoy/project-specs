# Project invariants (the constitution)

Immutable rules. Any code, document, or agent session violating them is wrong, even if the result works. Changing an invariant requires an ADR and an explanation of why the world changed.

## Privacy and access

1. **Isolation at input only.** Passing restricted facts to an LLM with an instruction "don't mention this" is forbidden. The audience filter is deterministic code before the model; the model never sees what it must not see.
2. **Fail-closed.** A fact without declared rights is visible to the owner only.
3. **Children's secrets are never retold to adults; adults' secrets are never revealed to children.** Retelling happens only at the owner's will (`sharing: hold` + `disclosure_hint`). The secret's owner is notified when someone else asks about it.
4. **The single exception is safety**: `safety_topics` (self-harm, violence, threat to life) break privacy and notify guardians. The exception is explicit, rare, and publicly documented in the charter.
5. **Every assembly and every write goes to audit.** An answer without a verifiable "what was included and why" does not conform.

## Data

6. **The data belongs to the family.** Full export (spec-compliant JSON + md projection) and full deletion — any time, no support ticket.
7. **Self-hosting is a day-one right.** The open server runs at home with full functionality; the cloud sells the absence of ops, not features.
8. **Cloud — data in Russia, 152-FZ compliance.**

## Architecture

9. **The brain never calls an LLM.** Context assembly is deterministic code; the model is chosen and paid for by the client. The brain is provider-agnostic.
10. **The project DB is the only source of truth; all writers go through the API.** Channels and UI store no memory.
11. **Model routing is per actor** (everyone pays for their own model, or a shared family key with a limit).
12. **One clarifying question per turn** — the limit of user patience.

## Licensing and brand

13. Format spec — MIT; documentation — CC-BY-4.0; core, bot, UI — AGPL-3.0; SDK and MCP — MIT; cloud and add-on services — proprietary. Brand and user data are not opened.
14. Format changes go through a spec change (semver) with example migration; code follows the spec, not the other way around.
