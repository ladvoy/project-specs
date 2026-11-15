# ADR-0003: Actor model and input-side audience filtering

Status: accepted (2026-11-15)

## Context
The whole family uses the base with different rights (children's secrets, adults' medical facts, wishlist vs diary). Asking an LLM "don't mention this" is unreliable: leaks under roleplay/injection, and the model is chosen by the user (including weak local ones).

## Decision
- Every request carries `X-Brain-Actor`; rights are enforced by deterministic assembly code **before** the LLM.
- A fact carries its `audience` (fail-closed: none — owner only); for those failing the gate — a `child_view` substitution, if defined.
- The constant core is assembled for the actor's role.
- Responses pass an optional leak-scan (literal strings of denied facts) for critical facts.
- Every write through the API is tagged with the source actor; facts from a child's dialogue default to `owner`.

## Consequences
+ Isolation is invisible as "restriction": each actor gets their own coherent world (child_view).
− Actor identification depends on the channel (TG binding/PIN) — an honest limit: the family contract + PIN.
