# Constraints, Indexes, and Full-Text Search

## What are Constraints?

**Constraints** are rules enforced by MySQL on the data in a column or table. They protect data integrity at the database level — not just in the application.

```
Application Layer   →  Validates user input (first line of defense)
Database Layer      →  Enforces constraints (last line of defense)
```

Both layers are necessary. An application bug, direct SQL access, or a data migration script can bypass application validation — but the database constraint always fires.

---

## Constraint Types

### 1. PRIMARY KEY

Uniquely identifies every row in a table.

```sql
CREATE TABLE customers (
  id BIGINT PRIMARY KEY AUTO_INCREMENT
);
```

- Automatically creates a **clustered index**
- Values must be **unique** and **NOT NULL**
- Only **one** primary key per table (may be composite)

### 2. FOREIGN KEY

Enforces a **referential integrity** relationship between two tables.

```sql
CREATE TABLE orders (
  id          BIGINT PRIMARY KEY AUTO_INCREMENT,
  customer_id BIGINT NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers(id)
    ON DELETE RESTRICT    -- prevent deleting a customer who has orders
    ON UPDATE CASCADE     -- update orders if customer PK changes
);
```

### Referential Actions

| Action | Behavior |
|--------|---------|
| `RESTRICT` | Block the parent DELETE/UPDATE if child rows exist |
| `CASCADE` | Automatically DELETE/UPDATE child rows when parent changes |
| `SET NULL` | Set the FK column to NULL when parent is deleted/updated |
| `NO ACTION` | Like RESTRICT (checked at end of statement) |
| `SET DEFAULT` | Set to the column's default value (rarely used) |

### 3. UNIQUE

Prevents duplicate values in a column (or combination of columns). Unlike PRIMARY KEY, allows one `NULL` value.

```sql
CREATE TABLE users (
  id    BIGINT       PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(120) NOT NULL UNIQUE,   -- single column
  username VARCHAR(50) NOT NULL
);

-- Composite unique constraint
ALTER TABLE users
  ADD CONSTRAINT uq_users_username UNIQUE (username);
```

### 4. NOT NULL

Requires a column to always have a value.

```sql
CREATE TABLE products (
  name  VARCHAR(120) NOT NULL,  -- required field
  notes TEXT                    -- optional field (NULL allowed)
);
```

### 5. DEFAULT

Provides a fallback value when no value is given on INSERT.

```sql
CREATE TABLE orders (
  status     VARCHAR(20)  NOT NULL DEFAULT 'pending',
  created_at TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
  is_paid    BOOLEAN      NOT NULL DEFAULT 0
);
```

### 6. CHECK (MySQL 8.0.16+)

Validates that column values satisfy a condition.

```sql
CREATE TABLE products (
  price    DECIMAL(12, 2) NOT NULL CHECK (price > 0),
  discount DECIMAL(5, 2)  NOT NULL DEFAULT 0 CHECK (discount BETWEEN 0 AND 100),
  stock    INT            NOT NULL CHECK (stock >= 0)
);
```

> **Note**: In older MySQL versions (< 8.0.16), `CHECK` is accepted by the parser but not enforced. Always verify your MySQL version.

---

## Complete Table Example with All Constraints

```sql
CREATE TABLE order_items (
  id         BIGINT         PRIMARY KEY AUTO_INCREMENT,
  order_id   BIGINT         NOT NULL,
  product_id BIGINT         NOT NULL,
  quantity   INT            NOT NULL DEFAULT 1 CHECK (quantity > 0),
  unit_price DECIMAL(12, 2) NOT NULL CHECK (unit_price > 0),
  discount   DECIMAL(5, 2)  NOT NULL DEFAULT 0 CHECK (discount BETWEEN 0 AND 100),
  UNIQUE (order_id, product_id),                              -- composite unique
  FOREIGN KEY (order_id)   REFERENCES orders(id)   ON DELETE CASCADE,
  FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE RESTRICT
);
```

---

## Adding and Removing Constraints

```sql
-- Add a FOREIGN KEY after table creation
ALTER TABLE orders
  ADD CONSTRAINT fk_orders_customer
  FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE CASCADE;

-- Remove a FOREIGN KEY
ALTER TABLE orders
  DROP FOREIGN KEY fk_orders_customer;

-- Add a UNIQUE constraint
ALTER TABLE customers
  ADD CONSTRAINT uq_customers_email UNIQUE (email);

-- Remove a UNIQUE constraint
ALTER TABLE customers
  DROP INDEX uq_customers_email;
```

---

## Indexes

**Indexes** are data structures that allow MySQL to find rows quickly without scanning the entire table. They dramatically speed up `WHERE`, `JOIN`, and `ORDER BY` operations.

### How an Index Works

```
Without index:
  MySQL scans all 5,000,000 rows → slow

With index on (email):
  MySQL jumps directly to matching rows → fast
```

The default index type in InnoDB is a **B-Tree** — balanced tree structure providing O(log n) lookup time.

### Index Types

| Type | Best For |
|------|---------|
| B-Tree (default) | Equality, range, prefix, ORDER BY |
| Hash | Exact equality (MEMORY engine only) |
| FULLTEXT | Natural language text search |
| SPATIAL | Geometry/location data |

### Creating Indexes

```sql
-- Single-column index
CREATE INDEX idx_customers_email ON customers(email);

-- Composite index (most useful for multi-column filtering)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);

-- Unique index (same as UNIQUE constraint)
CREATE UNIQUE INDEX uq_products_sku ON products(sku);
```

### Composite Index Column Order

Order matters for composite indexes. MySQL uses the **leftmost prefix rule**:

```sql
-- Index: (customer_id, created_at)
-- This index helps queries that filter by:
--   ✅ customer_id alone
--   ✅ customer_id + created_at together
--   ❌ created_at alone (index is NOT used)

SELECT * FROM orders WHERE customer_id = 5;                            -- ✅ uses index
SELECT * FROM orders WHERE customer_id = 5 AND created_at > '2026-01-01'; -- ✅ uses index
SELECT * FROM orders WHERE created_at > '2026-01-01';                  -- ❌ full scan
```

### When to Add Indexes

Add indexes on columns that are:
- ✅ Used in `WHERE` clauses
- ✅ Used in `JOIN` conditions
- ✅ Used in `ORDER BY` or `GROUP BY`
- ✅ Frequently searched with range conditions (`>`, `<`, `BETWEEN`)

### When NOT to Add Indexes

- ❌ Columns rarely used in queries
- ❌ Columns with very low cardinality (e.g., `is_active` with only 0 and 1)
- ❌ Tables with very few rows (< 1,000 rows — full scan is fast enough)
- ❌ Write-heavy tables where the overhead outweighs the read benefit

### Viewing and Dropping Indexes

```sql
-- View all indexes on a table
SHOW INDEX FROM orders;

-- Drop an index
DROP INDEX idx_orders_customer_date ON orders;
```

### Query Optimization with EXPLAIN

`EXPLAIN` shows how MySQL executes a query — whether it uses an index or does a full table scan.

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 5;
```

Key columns in EXPLAIN output:

| Column | What to Look For |
|--------|-----------------|
| `type` | `ref`, `range`, `const` = good; `ALL` = full scan (bad for large tables) |
| `key` | The index MySQL chose to use |
| `rows` | Estimated rows scanned — lower is better |
| `Extra` | `Using index` = very efficient; `Using filesort` = sort without index |

---

## Full-Text Search

Full-text search is optimized for searching natural language text — much more powerful than `LIKE '%keyword%'` on large text columns.

### Setup

```sql
-- Add a FULLTEXT index on title and body columns
ALTER TABLE articles
  ADD FULLTEXT INDEX idx_ft_title_body (title, body);
```

### Natural Language Mode (Default)

```sql
SELECT id, title,
  MATCH(title, body) AGAINST('mysql performance tuning') AS relevance
FROM articles
WHERE MATCH(title, body) AGAINST('mysql performance tuning')
ORDER BY relevance DESC;
```

### Boolean Mode

```sql
-- Must contain 'mysql', optional 'performance', must NOT contain 'deprecated'
SELECT id, title
FROM articles
WHERE MATCH(title, body)
      AGAINST('+mysql performance -deprecated' IN BOOLEAN MODE);
```

### LIKE vs FULLTEXT

| Aspect | `LIKE '%keyword%'` | FULLTEXT |
|--------|--------------------|---------|
| Index | ❌ Cannot use an index | ✅ Uses FULLTEXT index |
| Performance on large tables | Very slow | Fast |
| Relevance ranking | ❌ None | ✅ Built-in relevance score |
| Stemming / partial words | ❌ No | ✅ Yes (natural language mode) |
| Best for | Short strings, exact substrings | Long text, articles, search features |

---

## Best Practices

- Define constraints in schema design, not as an afterthought.
- Apply the **principle of least privilege**: use `RESTRICT` as the default FK action unless `CASCADE` is clearly correct.
- Index based on **actual query patterns** — don't guess, use `EXPLAIN` to validate.
- Review index overhead on write-heavy tables — every index slows down `INSERT`, `UPDATE`, and `DELETE`.
- Use FULLTEXT only when users genuinely need keyword search on large text columns.

## Common Mistakes

- Trusting only application validation — bypass is always possible through direct DB access.
- Creating too many indexes — they slow down writes and use disk space.
- Assuming every slow query is solved by adding an index — sometimes the query logic is the problem.
- Using `LIKE '%keyword%'` for search on large text columns — always causes a full table scan.

## Next Step

Continue to [11-table-relationship.md](11-table-relationship.md).

