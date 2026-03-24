# Database Design: Multi-Language System

## What We Are Building

A **multi-language (i18n) database** stores human-readable content — product names, descriptions, category labels, or any user-facing text — in multiple languages, served dynamically based on the requesting user's locale.

You have seen it in e-commerce:

```
🌐  Language: Bahasa Indonesia

  Produk       : Sepatu Lari Premium
  Kategori     : Olahraga & Kesehatan
  Deskripsi    : Sepatu lari ringan dengan sol responsif...

🌐  Language: English

  Product      : Premium Running Shoes
  Category     : Sports & Health
  Description  : Lightweight running shoes with a responsive sole...
```

Both pages query the **same rows** — only the translations differ.

---

## Why This Is More Complex Than It Looks

| Challenge | Description |
|-----------|-------------|
| **Schema design** | Where do we store translations? Inline columns don't scale |
| **Fallback logic** | What happens when a translation is missing for a language? |
| **Querying** | Fetching translated content efficiently with joins |
| **Maintenance** | Adding a new language or updating a translation |
| **URL slugs** | Slugs (used in URLs) may also be language-specific |
| **Consistency** | Ensuring all translatable entities have a translation for required languages |

---

## Common Approaches Compared

### Approach A: Inline Columns (Anti-pattern)

```sql
CREATE TABLE products (
    id          INT     NOT NULL AUTO_INCREMENT,
    name_en     VARCHAR(255),
    name_id     VARCHAR(255),
    name_fr     VARCHAR(255),
    -- one column per language
    PRIMARY KEY (id)
);
```

**Problems**:
- Adding a new language requires an `ALTER TABLE` on every translatable table
- Most columns are NULL for any given language
- Cannot query "all products missing a French translation" without ugly multi-column logic

---

### Approach B: JSON Column

```sql
CREATE TABLE products (
    id    INT  NOT NULL AUTO_INCREMENT,
    name  JSON NOT NULL,  -- {"en": "Shoes", "id": "Sepatu"}
    PRIMARY KEY (id)
);
```

**Acceptable for small projects.** But:
- Cannot create a standard index on a JSON key (requiring generated columns for indexing)
- Validating the presence of a translation is harder
- Bulk translation updates require JSON manipulation

---

### Approach C: Separate Translation Table (Chosen)

Each translatable text field lives in a dedicated `_translations` table:

```
products (1 row)  →  product_translations (N rows, one per language)
```

**Pros**:
- Add a new language without touching the main table
- `LEFT JOIN` fetches translated content in one query
- Easy to report which languages are missing translations
- Standard industry pattern (used by Magento, Shopware, Sylius, and many ORMs)

---

## What You Will Build in This Module

A multi-language product catalog database:

```
┌───────────────────────────────────────────────────────────────┐
│               Multi-Language Product Catalog                  │
│                                                               │
│  languages ─────────────────────────────────────┐            │
│     (en, id, fr, ja…)                            │            │
│                                                  │            │
│  categories ──────── category_translations ◄─────┤            │
│     │                 (name, slug per lang)      │            │
│     │                                            │            │
│  products   ──────── product_translations  ◄─────┘            │
│                       (name, desc, slug per lang)              │
└───────────────────────────────────────────────────────────────┘
```

---

## Learning Path

| Step | File | Topic |
|------|------|-------|
| 1 | `1-intro.md` | Overview, design approaches, schema map |
| 2 | `2-requirements.md` | Functional & non-functional requirements |
| 3 | `3-table-schema.md` | Full DDL — languages, categories, products, translations |
| 4 | `4-sql-implementation.md` | Seeding, querying, fallback logic, practical patterns |
