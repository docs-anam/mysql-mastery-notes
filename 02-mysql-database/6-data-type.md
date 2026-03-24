# MySQL Data Types

## Overview

Every column in a MySQL table must have a **data type** that defines:
- What kind of data it stores
- How much disk space it uses
- What operations are valid on it

Choosing the right type is one of the most important decisions in schema design — it affects **storage efficiency**, **query performance**, and **data validation**.

---

## 1. Numeric Types

### Integer Types

| Type | Storage | Signed Range | Unsigned Range | Typical Use |
|------|---------|-------------|----------------|-------------|
| `TINYINT` | 1 byte | -128 to 127 | 0 to 255 | Flags, booleans, status codes |
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | 0 to 65,535 | Ages, small quantities |
| `MEDIUMINT` | 3 bytes | -8M to 8M | 0 to 16M | Mid-range counters |
| `INT` / `INTEGER` | 4 bytes | -2.1B to 2.1B | 0 to 4.3B | IDs, general counts ✅ |
| `BIGINT` | 8 bytes | ±9.2 × 10¹⁸ | 0 to 1.8 × 10¹⁹ | Large IDs, timestamps in ms |

```sql
CREATE TABLE users (
  id         BIGINT   PRIMARY KEY AUTO_INCREMENT,  -- large ID space
  age        TINYINT  UNSIGNED,                    -- 0–255
  post_count INT      NOT NULL DEFAULT 0            -- regular counter
);
```

### Fixed-Precision Decimal

| Type | Description | Use Case |
|------|-------------|---------|
| `DECIMAL(p, s)` | Exact precision; `p` total digits, `s` after decimal | Money, financial data ✅ |
| `NUMERIC(p, s)` | Alias for `DECIMAL` | Same as above |

```sql
CREATE TABLE products (
  price    DECIMAL(12, 2) NOT NULL,  -- up to 9,999,999,999.99
  tax_rate DECIMAL(5, 4)             -- e.g., 0.1100 = 11%
);
```

> **Always use `DECIMAL` for money** — `FLOAT` and `DOUBLE` use binary floating-point representation, which causes rounding errors like `0.1 + 0.2 = 0.30000000000000004`.

### Approximate Floating Point

| Type | Storage | Precision | Use Case |
|------|---------|-----------|---------|
| `FLOAT` | 4 bytes | ~7 digits | Scientific measurements (not money) |
| `DOUBLE` | 8 bytes | ~15 digits | High-precision scientific data |

---

## 2. String Types

### Fixed and Variable Length

| Type | Description | Max Size | Use Case |
|------|-------------|---------|---------|
| `CHAR(n)` | Fixed length — always uses `n` bytes | 255 chars | Country codes, fixed codes (`'ID'`, `'US'`) |
| `VARCHAR(n)` | Variable length — uses only needed bytes | 65,535 chars | Names, emails, titles ✅ |
| `TINYTEXT` | Very small text | 255 bytes | Short notes |
| `TEXT` | Medium text | 65,535 bytes | Articles, descriptions |
| `MEDIUMTEXT` | Large text | 16 MB | Long content, HTML |
| `LONGTEXT` | Very large text | 4 GB | Logs, large documents |

```sql
CREATE TABLE articles (
  id           BIGINT       PRIMARY KEY AUTO_INCREMENT,
  country_code CHAR(2)      NOT NULL,            -- always exactly 2 chars
  title        VARCHAR(200) NOT NULL,            -- variable length
  body         TEXT         NOT NULL,            -- long content
  summary      VARCHAR(500)                      -- optional short summary
);
```

### CHAR vs VARCHAR

| Aspect | CHAR(10) | VARCHAR(10) |
|--------|----------|------------|
| `'hi'` stored | `'hi        '` (10 bytes) | `'hi'` (3 bytes: 2 + 1 length) |
| Storage | Fixed — always allocated | Variable — only what's needed |
| Speed | Slightly faster to read | Slightly slower (variable lookup) |
| Best for | Fixed-width codes | Most text data |

### Binary Types

| Type | Description | Use Case |
|------|-------------|---------|
| `BINARY(n)` | Fixed-length binary | Hash digests stored as bytes |
| `VARBINARY(n)` | Variable-length binary | Custom binary data |
| `BLOB` | Binary large object | Images, files (usually store path instead) |

### ENUM and SET

```sql
-- ENUM: column must be one of the listed values
CREATE TABLE products (
  status ENUM('active', 'inactive', 'discontinued') NOT NULL DEFAULT 'active'
);

-- SET: column can contain zero or more of the listed values
CREATE TABLE users (
  permissions SET('read', 'write', 'delete', 'admin')
);
```

> Use `ENUM` for **stable, low-cardinality values** only. Avoid it for values that change frequently — adding a new option requires an `ALTER TABLE`.

---

## 3. Date and Time Types

| Type | Format | Storage | Use Case |
|------|--------|---------|---------|
| `DATE` | `YYYY-MM-DD` | 3 bytes | Birthdays, event dates |
| `TIME` | `HH:MM:SS` | 3–6 bytes | Duration, time of day |
| `YEAR` | `YYYY` | 1 byte | Year-only values |
| `DATETIME` | `YYYY-MM-DD HH:MM:SS` | 5–8 bytes | Meeting times, scheduled events |
| `TIMESTAMP` | `YYYY-MM-DD HH:MM:SS` | 4–7 bytes | Audit timestamps (`created_at`, `updated_at`) |

### DATETIME vs TIMESTAMP

| Feature | DATETIME | TIMESTAMP |
|---------|---------|----------|
| Range | 1000-01-01 to 9999-12-31 | 1970-01-01 to 2038-01-19 |
| Timezone aware | ❌ No — stores literal value | ✅ Yes — converts to/from UTC |
| Storage | 5 bytes | 4 bytes |
| Default `CURRENT_TIMESTAMP` | ✅ Supported (MySQL 5.6+) | ✅ Supported |
| Best for | Historical/business dates | Audit fields — `created_at`, `updated_at` |

```sql
CREATE TABLE orders (
  id           BIGINT    PRIMARY KEY AUTO_INCREMENT,
  order_date   DATE      NOT NULL,                                -- just the date
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,     -- auto-set on insert
  updated_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP      -- auto-update on change
                         ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 4. Boolean

MySQL does **not** have a native boolean type. `BOOLEAN` is an alias for `TINYINT(1)`.

```sql
CREATE TABLE products (
  is_active  BOOLEAN     NOT NULL DEFAULT 1,  -- 1 = true, 0 = false
  is_deleted TINYINT(1)  NOT NULL DEFAULT 0
);

-- Query
SELECT * FROM products WHERE is_active = 1;
SELECT * FROM products WHERE is_active = TRUE;  -- both work
```

---

## 5. JSON Type (MySQL 5.7.8+)

Stores structured JSON data with native parsing and indexing support.

```sql
CREATE TABLE products (
  id       BIGINT PRIMARY KEY AUTO_INCREMENT,
  name     VARCHAR(120) NOT NULL,
  metadata JSON
);

-- Insert with JSON
INSERT INTO products (name, metadata)
VALUES ('Laptop', '{"brand": "Dell", "ram_gb": 16, "tags": ["sale", "new"]}');

-- Query JSON fields
SELECT name, metadata->>'$.brand' AS brand
FROM products
WHERE metadata->>'$.ram_gb' > 8;
```

> Use `JSON` when structure varies per row. For structured, queryable data, prefer proper normalized columns with correct types.

---

## Complete Schema Example

```sql
CREATE TABLE products (
  id           BIGINT         PRIMARY KEY AUTO_INCREMENT,
  sku          VARCHAR(40)    NOT NULL UNIQUE,
  name         VARCHAR(120)   NOT NULL,
  description  TEXT,
  price        DECIMAL(12, 2) NOT NULL,
  stock        INT            NOT NULL DEFAULT 0,
  status       ENUM('active', 'inactive', 'discontinued') NOT NULL DEFAULT 'active',
  is_featured  BOOLEAN        NOT NULL DEFAULT 0,
  metadata     JSON,
  created_at   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Type Selection Quick Reference

| Data | Recommended Type |
|------|-----------------|
| Auto-incrementing ID | `BIGINT UNSIGNED AUTO_INCREMENT` |
| Money / price | `DECIMAL(10, 2)` |
| Short text (email, name) | `VARCHAR(n)` |
| Long text (articles) | `TEXT` |
| Fixed code (country, currency) | `CHAR(n)` |
| True/False flag | `BOOLEAN` (`TINYINT(1)`) |
| Status with few options | `ENUM(...)` |
| Date only | `DATE` |
| Audit timestamp | `TIMESTAMP DEFAULT CURRENT_TIMESTAMP` |
| Flexible structured data | `JSON` |

---

## Best Practices

- Use the **smallest type** that safely fits your data range — saves storage and speeds up scans.
- **Always** use `DECIMAL` for monetary values — never `FLOAT` or `DOUBLE`.
- Use `VARCHAR` for most text; only use `TEXT` when content genuinely exceeds ~200 characters.
- Use `utf8mb4` character set on text columns to safely store all Unicode characters.
- Use `TIMESTAMP` for `created_at`/`updated_at` audit fields with `DEFAULT CURRENT_TIMESTAMP`.

## Common Mistakes

- Using `FLOAT` for money — produces silent rounding errors in calculations.
- Using `TEXT` when `VARCHAR` is sufficient — `TEXT` columns can't have default values and index limitations.
- Using `ENUM` for values that change frequently — each change requires `ALTER TABLE`.
- Storing boolean as `'yes'/'no'` strings instead of `TINYINT(1)`.

## Next Step

Continue to [7-table.md](7-table.md).

