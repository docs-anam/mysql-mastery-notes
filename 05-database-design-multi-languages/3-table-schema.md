# Table Schema

## Create the Database

```sql
CREATE DATABASE IF NOT EXISTS multilang_catalog
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE multilang_catalog;
```

> `utf8mb4` is mandatory here. Translations into Japanese, Arabic, Chinese, or any emoji-containing language require the full 4-byte Unicode range that `utf8mb4` provides. Legacy `utf8` in MySQL only covers 3-byte Unicode and will silently corrupt or reject 4-byte characters.

---

## 1. `languages`

The language registry. Every translation row references a row in this table.

```sql
CREATE TABLE languages (
    id         TINYINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    code       VARCHAR(10)       NOT NULL,   -- IETF BCP 47: 'en', 'id', 'fr', 'zh-CN'
    name       VARCHAR(100)      NOT NULL,   -- 'English', 'Bahasa Indonesia', 'Français'
    name_local VARCHAR(100)      NOT NULL,   -- 'English', 'Bahasa Indonesia', 'Français' (in the language itself)
    is_default BOOLEAN           NOT NULL DEFAULT FALSE,
    is_active  BOOLEAN           NOT NULL DEFAULT TRUE,
    sort_order TINYINT UNSIGNED  NOT NULL DEFAULT 0,

    PRIMARY KEY (id),
    UNIQUE KEY uq_lang_code       (code),
    INDEX       idx_lang_active   (is_active),
    INDEX       idx_lang_default  (is_default)
) ENGINE=InnoDB;
```

### Column Guide

| Column | Purpose |
|--------|---------|
| `code` | IETF locale code used by the application and stored in translation rows |
| `name` | The language name in English — for admin UIs |
| `name_local` | The language name in its own script — for language-switcher UIs |
| `is_default` | Exactly one row should be `TRUE`; application falls back to this language |
| `is_active` | `FALSE` hides a language from the switcher without data loss |
| `sort_order` | Controls display order in drop-downs |

### Enforcing a Single Default Language

MySQL does not support a `CHECK (COUNT(*) WHERE is_default) = 1` constraint. Enforce this at the application layer or via a trigger:

```sql
DELIMITER $$
CREATE TRIGGER trg_one_default_language
BEFORE INSERT ON languages
FOR EACH ROW
BEGIN
    IF NEW.is_default = TRUE THEN
        UPDATE languages SET is_default = FALSE WHERE is_default = TRUE;
    END IF;
END $$
DELIMITER ;
```

The same trigger logic applies to `BEFORE UPDATE`.

---

## 2. `categories`

Stores only the **non-translatable** fields of a category.

```sql
CREATE TABLE categories (
    id         INT UNSIGNED      NOT NULL AUTO_INCREMENT,
    parent_id  INT UNSIGNED      NULL,
    image_url  VARCHAR(512)      NULL,
    sort_order SMALLINT UNSIGNED NOT NULL DEFAULT 0,
    is_active  BOOLEAN           NOT NULL DEFAULT TRUE,
    created_at DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    INDEX idx_cat_parent  (parent_id),
    INDEX idx_cat_active  (is_active),
    CONSTRAINT fk_cat_parent
        FOREIGN KEY (parent_id) REFERENCES categories (id)
        ON DELETE SET NULL
) ENGINE=InnoDB;
```

### Design Decisions

| Decision | Reason |
|----------|--------|
| `parent_id` self-reference | Allows nested category trees (e.g., Sports → Running → Road Running) |
| `ON DELETE SET NULL` | Deleting a parent category orphans children to the root rather than cascading deletes |
| No `name` column | All label data lives in `category_translations` |

---

## 3. `category_translations`

One row per language per category.

```sql
CREATE TABLE category_translations (
    id          INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    category_id INT UNSIGNED  NOT NULL,
    locale      VARCHAR(10)   NOT NULL,
    name        VARCHAR(255)  NOT NULL,
    slug        VARCHAR(255)  NOT NULL,
    description TEXT          NULL,

    PRIMARY KEY (id),
    UNIQUE KEY uq_cat_trans_locale     (category_id, locale),
    UNIQUE KEY uq_cat_trans_slug_locale (locale, slug),
    INDEX       idx_cat_trans_locale   (locale),

    CONSTRAINT fk_ct_category
        FOREIGN KEY (category_id) REFERENCES categories (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_ct_locale
        FOREIGN KEY (locale)      REFERENCES languages (code)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
) ENGINE=InnoDB;
```

### Unique Constraint Breakdown

| Constraint | Purpose |
|-----------|---------|
| `uq_cat_trans_locale (category_id, locale)` | One translation per language per category |
| `uq_cat_trans_slug_locale (locale, slug)` | Slugs are unique within a language (two categories cannot share `/sports` in English) |

### `FK … ON UPDATE CASCADE`

When a locale code changes (e.g., `'zh'` → `'zh-CN'`), the `ON UPDATE CASCADE` on `locale` propagates the change automatically to all translation rows. Without it, you'd have to manually update every translation row first.

---

## 4. `products`

Stores only the **non-translatable** fields of a product.

```sql
CREATE TABLE products (
    id          BIGINT UNSIGNED   NOT NULL AUTO_INCREMENT,
    category_id INT UNSIGNED      NOT NULL,
    sku         VARCHAR(100)      NOT NULL,
    price       DECIMAL(15, 2)    NOT NULL,
    stock       INT UNSIGNED      NOT NULL DEFAULT 0,
    weight_gram INT UNSIGNED      NULL,
    image_url   VARCHAR(512)      NULL,
    is_active   BOOLEAN           NOT NULL DEFAULT TRUE,
    created_at  DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_product_sku    (sku),
    INDEX       idx_product_cat  (category_id),
    INDEX       idx_product_active (is_active),

    CONSTRAINT fk_prod_category
        FOREIGN KEY (category_id) REFERENCES categories (id)
) ENGINE=InnoDB;
```

### Why `DECIMAL(15, 2)` for Price?

| Type | Problem |
|------|---------|
| `FLOAT` / `DOUBLE` | Binary floating-point cannot represent many decimal fractions exactly. `0.1 + 0.2 ≠ 0.3` |
| `INT` (store cents) | Acceptable, but requires unit conversion at read/write; `DECIMAL` is clearer |
| `DECIMAL(15, 2)` | Exact decimal arithmetic. Up to 999,999,999,999,999.99 — no rounding errors |

---

## 5. `product_translations`

One row per language per product.

```sql
CREATE TABLE product_translations (
    id          BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    product_id  BIGINT UNSIGNED  NOT NULL,
    locale      VARCHAR(10)      NOT NULL,
    name        VARCHAR(255)     NOT NULL,
    slug        VARCHAR(255)     NOT NULL,
    description TEXT             NULL,

    PRIMARY KEY (id),
    UNIQUE KEY uq_prod_trans_locale      (product_id, locale),
    UNIQUE KEY uq_prod_trans_slug_locale (locale, slug),
    INDEX       idx_prod_trans_locale    (locale),

    CONSTRAINT fk_pt_product
        FOREIGN KEY (product_id) REFERENCES products (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_pt_locale
        FOREIGN KEY (locale)     REFERENCES languages (code)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
) ENGINE=InnoDB;
```

---

## Full Schema Diagram

```
languages
─────────────────────────
id  (PK)
code  (UNIQUE)               ◄──── category_translations.locale
name                         ◄──── product_translations.locale
name_local
is_default
is_active


categories                    category_translations
──────────────────────        ──────────────────────────────
id  (PK)              ◄────── category_id
parent_id ──► self            locale ──────────► languages.code
image_url                     name
sort_order                    slug
is_active                     description


products                      product_translations
──────────────────────        ──────────────────────────────
id  (PK)              ◄────── product_id
category_id ──► categories    locale ──────────► languages.code
sku  (UNIQUE)                 name
price                         slug
stock                         description
is_active
```
