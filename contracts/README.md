# Межреповые контракты

Источник правды для интерфейсов между компонентами. SDK генерируются из этих контрактов; CI проверяет совместимость (openapi-diff). Меняешь контракт — PR здесь, потом реализации.

## Brain API (v0)

Headless, provider-agnostic (мозг не вызывает LLM). OpenAPI: `contracts/brain.openapi.yaml` (генерируется из кода `brain`, вендорится сюда при релизе).

- **Management-плоскость** (`/admin/*`, project-token): проекты, акторы, сущности, факты, связи, proposals, audit.
- **Query-плоскость** (`/context/*`, context-token + `X-Brain-Actor`): сборка контекста. Ответ = `context` + обязательный `audit` (included / excluded+reason / substitutions).

## MCP-инструменты (brain-mcp → brain API)

| Инструмент | Endpoint | Семантика |
|---|---|---|
| `get_context(query, intent?)` | POST `/context` | actor из привязки; ответ — собранный блок с цитатами |
| `get_portrait(name)` | GET `/entities?alias=` | карточка сущности |
| `propose_memory(entity, text, source)` | POST facts (`status: pending`) | кандидат; подтверждение владельцем |
| `search_audit(...)` | GET `/audit` | журнал (для владельцев) |

**Правило для описаний инструментов:** «вызывай при любом упоминании людей из семьи, подарков, здоровья, планов» — навигация модели задаётся description'ом, не логикой ядра.

## Actor-привязка (каналы)

- Telegram: `telegram_user_id` ↔ actor slug, привязка при онбординге;-child-профиль — PIN.
- Web (Next.js): login аккаунта = actor; детский — PIN.
- Brain доверяет только своей таблице привязок; канал не может «назначить» actor, которого нет в проекте.

## Формат портретов

Внешний стандарт — репо `portrait-spec` (MIT). Ядро хранит факты в форме спеки v0.2; изменение формата — change в `portrait-spec` с миграцией примеров.
