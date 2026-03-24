# Requirement Specification

Before writing a single `CREATE TABLE`, we must understand what the system needs to do.

---

## Actors

```
┌──────────────┐   requests page in locale   ┌──────────────────┐
│  Customer    │ ───────────────────────────►│  Application     │
│  (browser /  │                             │  (backend API)   │
│   mobile)    │ ◄───────────────────────────│                  │
└──────────────┘   responds with translated  └──────┬───────────┘
                   content                          │ queries with locale
                                                    ▼
                                            ┌──────────────────┐
┌──────────────┐   manages translations     │  Database        │
│  Admin /     │ ──────────────────────────►│  (this module)   │
│  Translator  │                            └──────────────────┘
└──────────────┘
```

| Actor | Role |
|-------|------|
| **Customer** | Browses the catalog; requests content in their preferred language |
| **Application** | Passes a locale code when querying; handles fallback to default language |
| **Admin / Translator** | Inserts and updates translation rows for each language |
| **Database** | Stores all languages, content, and translations; enforces referential integrity |

---

## Functional Requirements

### FR-1: Language Registry

- The system must maintain a registry of supported languages.
- Each language has a unique **locale code** (e.g., `en`, `id`, `fr`, `ja`).
- Exactly **one** language must be designated as the **default** language.
- Languages can be **activated or deactivated** without deletion.

### FR-2: Translatable Entities

For this module, the translatable entities are **categories** and **products**:

| Entity | Non-translatable fields | Translatable fields |
|--------|------------------------|---------------------|
| `categories` | `id`, `parent_id`, `image_url`, `sort_order`, `is_active` | `name`, `slug`, `description` |
| `products` | `id`, `category_id`, `sku`, `price`, `stock`, `is_active` | `name`, `slug`, `description` |

### FR-3: Translation Storage

- Each translatable entity must have a corresponding `*_translations` table.
- A translation row stores the locale code, the entity ID, and the translatable fields.
- Each `(entity_id, locale)` pair must be unique — no duplicate translations per language.

### FR-4: Querying with Fallback

- When querying content in a language, the application must receive the translation for that language **if it exists**.
- If a translation does **not** exist for the requested language, the system must fall back to the **default language** translation.
- This fallback must be implemented at the SQL layer using `COALESCE` or conditional joins — not in application code.

### FR-5: Slug Uniqueness per Language

- URL slugs (e.g., `running-shoes`, `sepatu-lari`) are language-specific and stored in the translation table.
- A slug must be **unique within its language** — two products may not share the same slug in the same language.
- Slugs from different languages for the same entity may differ freely.

### FR-6: Translation Completeness Reporting

- It must be possible to query which entities are **missing a translation** for a given language.
- This enables admin dashboards to show translation progress (e.g., "23 products have no French translation").

### FR-7: Adding a New Language

- Adding a new language must not require any schema change (`ALTER TABLE`).
- A new language is added by inserting a row into the `languages` table and then providing translation rows for each entity.

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| **Product page load** | Translation query response < 10ms with indexes in place |
| **Catalog list query** | 100 products in a language returned < 50ms |
| **Schema evolution** | Adding a new language requires zero DDL changes |
| **Data integrity** | FK constraints prevent orphaned translation rows |
| **Slug collisions** | Unique index prevents duplicate slugs per language |

---

## Out of Scope

The following are intentionally excluded from this module to keep the schema focused:

- Full-text search across languages
- Translation versioning or approval workflows
- Currency / price localisation (separate from language)
- Date / number format localisation (handled by the application layer)
