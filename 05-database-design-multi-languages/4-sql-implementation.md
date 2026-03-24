# SQL Implementation

## 1. Seed: Languages

```sql
INSERT INTO languages (code, name, name_local, is_default, is_active, sort_order) VALUES
('en',    'English',            'English',            TRUE,  TRUE, 1),
('id',    'Indonesian',         'Bahasa Indonesia',   FALSE, TRUE, 2),
('fr',    'French',             'Français',           FALSE, TRUE, 3),
('ja',    'Japanese',           '日本語',             FALSE, TRUE, 4);
```

> The `BEFORE INSERT` trigger defined in `3-table-schema.md` ensures only `en` ends up with `is_default = TRUE` — the trigger clears any previous default when a new one is declared.

---

## 2. Seed: Categories

```sql
-- Non-translatable data
INSERT INTO categories (id, parent_id, sort_order) VALUES
(1, NULL, 1),  -- Sports & Health (root)
(2, 1,    1),  -- Running (child of Sports)
(3, NULL, 2),  -- Electronics (root)
(4, 3,    1);  -- Smartphones (child of Electronics)

-- Translations
INSERT INTO category_translations (category_id, locale, name, slug, description) VALUES
-- Sports & Health
(1, 'en', 'Sports & Health',   'sports-health',   'All sports and fitness products'),
(1, 'id', 'Olahraga & Kesehatan', 'olahraga-kesehatan', 'Semua produk olahraga dan kebugaran'),
(1, 'fr', 'Sports & Santé',    'sports-sante',    'Tous les produits sportifs et fitness'),

-- Running (no French translation intentionally — used to demo fallback)
(2, 'en', 'Running',  'running',  'Road, trail, and track running products'),
(2, 'id', 'Lari',     'lari',     'Produk lari jalan, trail, dan lintasan'),

-- Electronics
(3, 'en', 'Electronics',   'electronics',   'Consumer electronics'),
(3, 'id', 'Elektronik',    'elektronik',    'Elektronik konsumen'),
(3, 'fr', 'Électronique',  'electronique',  'Électronique grand public'),

-- Smartphones
(4, 'en', 'Smartphones', 'smartphones', 'Mobile phones and accessories'),
(4, 'id', 'Ponsel Pintar', 'ponsel-pintar', 'Ponsel dan aksesori');
```

---

## 3. Seed: Products

```sql
-- Non-translatable data
INSERT INTO products (id, category_id, sku, price, stock, weight_gram) VALUES
(1, 2, 'SHOE-RUN-001', 899000.00, 150, 320),
(2, 2, 'SHOE-RUN-002', 1299000.00, 80,  295),
(3, 4, 'PHONE-X-001',  8999000.00, 40,  172),
(4, 4, 'PHONE-Y-002',  12499000.00, 25, 189);

-- Translations
INSERT INTO product_translations (product_id, locale, name, slug, description) VALUES
-- Product 1 (running shoe — entry level)
(1, 'en', 'AeroStride 200',
          'aerostride-200',
          'Lightweight everyday trainer with cushioned midsole. Ideal for beginners.'),
(1, 'id', 'AeroStride 200',
          'aerostride-200-id',
          'Sepatu latihan harian yang ringan dengan midsole bantalan. Ideal untuk pemula.'),
(1, 'fr', 'AeroStride 200',
          'aerostride-200-fr',
          'Chaussure de training légère avec semelle intermédiaire rembourrée. Idéale pour débutants.'),

-- Product 2 (running shoe — performance)
(2, 'en', 'AeroStride Pro',
          'aerostride-pro',
          'Competition-grade carbon-plate racer for sub-4-minute milers.'),
(2, 'id', 'AeroStride Pro',
          'aerostride-pro-id',
          'Sepatu balap pelat karbon kelas kompetisi untuk atlet sub-4 menit per kilometer.'),
-- No French translation for product 2 — intentional for fallback demo

-- Product 3 (smartphone)
(3, 'en', 'Lumina X1',
          'lumina-x1',
          '6.1" OLED display, 50 MP triple camera, 5000 mAh battery.'),
(3, 'id', 'Lumina X1',
          'lumina-x1-id',
          'Layar OLED 6.1 inci, kamera triple 50 MP, baterai 5000 mAh.'),

-- Product 4 (smartphone — flagship)
(4, 'en', 'Lumina X1 Ultra',
          'lumina-x1-ultra',
          '6.8" ProMotion LTPO display, 200 MP periscope zoom, titanium chassis.'),
(4, 'id', 'Lumina X1 Ultra',
          'lumina-x1-ultra-id',
          'Layar ProMotion LTPO 6.8 inci, zoom periskop 200 MP, rangka titanium.');
```

---

## 4. Fetching a Single Product with Fallback

The core query pattern: request a translation for a specific locale, fall back to the default language if unavailable.

```sql
-- Fetch product id=2 in French (fr), fallback to English (en)
SELECT
    p.id,
    p.sku,
    p.price,
    p.stock,
    -- Use the French translation if it exists, otherwise fall back to English
    COALESCE(pt_req.name,        pt_def.name)        AS name,
    COALESCE(pt_req.slug,        pt_def.slug)        AS slug,
    COALESCE(pt_req.description, pt_def.description) AS description
FROM products p

-- Join for the requested locale (outer join — may return NULL if no translation)
LEFT JOIN product_translations pt_req
    ON pt_req.product_id = p.id
   AND pt_req.locale     = 'fr'

-- Join for the default locale (always present for complete products)
LEFT JOIN product_translations pt_def
    ON pt_def.product_id = p.id
   AND pt_def.locale     = 'en'

WHERE p.id = 2
  AND p.is_active = TRUE;
```

**Result for product 2 in French (no French translation exists):**

```
id  | sku           | price      | stock | name          | slug          | description
----|---------------|------------|-------|---------------|---------------|----------------------------
 2  | SHOE-RUN-002  | 1299000.00 | 80    | AeroStride Pro | aerostride-pro | Competition-grade carbon...
```

The English fallback is transparently returned.

---

## 5. Fetching the Full Product Catalog

```sql
-- All active products in Indonesian, fallback to English
SELECT
    p.id,
    p.sku,
    p.price,
    p.stock,
    COALESCE(pt_req.name,        pt_def.name)        AS name,
    COALESCE(pt_req.slug,        pt_def.slug)        AS slug,
    COALESCE(pt_req.description, pt_def.description) AS description,
    COALESCE(ct_req.name,        ct_def.name)        AS category_name
FROM products p

LEFT JOIN product_translations pt_req
    ON pt_req.product_id = p.id AND pt_req.locale = 'id'
LEFT JOIN product_translations pt_def
    ON pt_def.product_id = p.id AND pt_def.locale = 'en'

JOIN categories c ON c.id = p.category_id

LEFT JOIN category_translations ct_req
    ON ct_req.category_id = c.id AND ct_req.locale = 'id'
LEFT JOIN category_translations ct_def
    ON ct_def.category_id = c.id AND ct_def.locale = 'en'

WHERE p.is_active = TRUE
ORDER BY p.id;
```

---

## 6. Fetching a Product by Slug

URLs are language-specific. Given `/id/aerostride-200-id`, find the product.

```sql
SELECT
    p.id,
    p.sku,
    p.price,
    pt.name,
    pt.description
FROM product_translations pt
JOIN products p ON p.id = pt.product_id
WHERE pt.locale = 'id'
  AND pt.slug   = 'aerostride-200-id'
  AND p.is_active = TRUE;
```

The `UNIQUE KEY uq_prod_trans_slug_locale (locale, slug)` guarantees this returns at most one row without a `LIMIT`.

---

## 7. Inserting / Updating a Translation

### Insert a new translation

```sql
INSERT INTO product_translations (product_id, locale, name, slug, description)
VALUES (2, 'fr', 'AeroStride Pro', 'aerostride-pro-fr',
        'Chaussure de course à plaque de carbone pour compétition.');
```

### Update an existing translation

```sql
UPDATE product_translations
SET    name        = 'AeroStride Pro Edition',
       description = 'Mise à jour: Plaque de carbone et semelle améliorées.'
WHERE  product_id  = 2
  AND  locale      = 'fr';
```

### Upsert (insert or update) using `ON DUPLICATE KEY UPDATE`

```sql
INSERT INTO product_translations (product_id, locale, name, slug, description)
VALUES (2, 'fr', 'AeroStride Pro', 'aerostride-pro-fr',
        'Chaussure de course à plaque de carbone.')
ON DUPLICATE KEY UPDATE
    name        = VALUES(name),
    description = VALUES(description);
-- slug intentionally excluded: if a slug was already set, don't overwrite it
-- (slug changes break existing URLs)
```

---

## 8. Translation Completeness Report

Find all products that are **missing a translation** for a given language — essential for translation management dashboards.

```sql
-- Products missing a Japanese (ja) translation
SELECT
    p.id,
    p.sku,
    pt_default.name AS name_in_default_language
FROM products p
LEFT JOIN product_translations pt_ja
    ON pt_ja.product_id = p.id
   AND pt_ja.locale     = 'ja'
LEFT JOIN product_translations pt_default
    ON pt_default.product_id = p.id
   AND pt_default.locale     = 'en'
WHERE pt_ja.id IS NULL     -- no Japanese translation row exists
  AND p.is_active = TRUE
ORDER BY p.id;
```

### Coverage Summary Across All Languages

```sql
-- Count translated vs total products per language
SELECT
    l.code,
    l.name                            AS language,
    COUNT(p.id)                       AS total_products,
    COUNT(pt.id)                      AS translated,
    COUNT(p.id) - COUNT(pt.id)        AS missing,
    ROUND(COUNT(pt.id) / COUNT(p.id) * 100, 1) AS coverage_pct
FROM languages l
CROSS JOIN products p
LEFT JOIN product_translations pt
    ON pt.product_id = p.id
   AND pt.locale     = l.code
WHERE l.is_active = TRUE
  AND p.is_active  = TRUE
GROUP BY l.code, l.name
ORDER BY l.sort_order;
```

**Example result:**

```
code | language          | total_products | translated | missing | coverage_pct
-----|-------------------|----------------|------------|---------|-------------
en   | English           | 4              | 4          | 0       | 100.0
id   | Indonesian        | 4              | 4          | 0       | 100.0
fr   | French            | 4              | 2          | 2       | 50.0
ja   | Japanese          | 4              | 0          | 4       | 0.0
```

---

## 9. Adding a New Language

No schema change required. Simply insert a new language row and start adding translation rows.

```sql
-- Step 1: Register the language
INSERT INTO languages (code, name, name_local, is_default, is_active, sort_order)
VALUES ('de', 'German', 'Deutsch', FALSE, TRUE, 5);

-- Step 2: Add translations as they become available
INSERT INTO product_translations (product_id, locale, name, slug, description)
VALUES
(1, 'de', 'AeroStride 200', 'aerostride-200-de',
    'Leichter Alltagstrainer mit gepolsterter Zwischensohle.'),
(3, 'de', 'Lumina X1', 'lumina-x1-de',
    '6,1" OLED-Display, 50-MP-Dreifachkamera, 5000-mAh-Akku.');
```

Until a translation exists, the fallback query in Section 4 will automatically serve English content.

---

## 10. Deleting a Language

```sql
-- The ON DELETE RESTRICT on FK prevents deletion if any translation rows still reference this locale.
-- First, remove all translations for that language:
DELETE FROM product_translations  WHERE locale = 'de';
DELETE FROM category_translations WHERE locale = 'de';

-- Then deactivate or delete the language row:
UPDATE languages SET is_active = FALSE WHERE code = 'de';
-- or hard delete:
DELETE FROM languages WHERE code = 'de';
```

> Prefer `is_active = FALSE` over hard deletion. It preserves the fact that translations once existed and allows recovery if the decision is reversed.

---

## Summary: Key Patterns

| Pattern | SQL Technique |
|---------|--------------|
| Fetch with fallback | Two `LEFT JOIN` to `*_translations` + `COALESCE` |
| Lookup by slug | Query `*_translations` where `locale = ? AND slug = ?` |
| Insert or update translation | `ON DUPLICATE KEY UPDATE` |
| Missing-translation report | `LEFT JOIN *_translations … WHERE pt.id IS NULL` |
| Coverage dashboard | `CROSS JOIN languages` + `LEFT JOIN translations` + `COUNT` |
| Add new language | `INSERT INTO languages` — zero DDL change |
