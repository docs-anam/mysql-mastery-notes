# Tables and CRUD

## What is a Table?

A **table** is the core structure in a relational database. It organizes data into rows (records) and columns (attributes). Every piece of data in MySQL lives inside a table.

```
┌─────────────────────────────────────────────────────────────┐
│                       customers                             │
├─────┬──────────────────┬──────────────────┬─────────────────┤
│ id  │ full_name        │ email            │ created_at      │
├─────┼──────────────────┼──────────────────┼─────────────────┤
│ 1   │ Ana Putri        │ ana@example.com  │ 2026-01-15 ...  │
│ 2   │ Budi Santoso     │ budi@example.com │ 2026-02-01 ...  │
│ 3   │ Citra Dewi       │ citra@mail.com   │ 2026-02-20 ...  │
└─────┴──────────────────┴──────────────────┴─────────────────┘
   ↑         ↑                  ↑
  PK      Column             Column
```

---

## Creating Tables

### Full Syntax

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
  column_name data_type [column_constraints],
  ...,
  [table_constraints]
);
```

### Example: Customers Table

```sql
CREATE TABLE customers (
  id         BIGINT        PRIMARY KEY AUTO_INCREMENT,
  full_name  VARCHAR(120)  NOT NULL,
  email      VARCHAR(120)  NOT NULL UNIQUE,
  phone      VARCHAR(20),
  is_active  BOOLEAN       NOT NULL DEFAULT 1,
  created_at TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Example: Many-to-Many Bridge Table (Composite Primary Key)

```sql
CREATE TABLE order_products (
  order_id   BIGINT NOT NULL,
  product_id BIGINT NOT NULL,
  quantity   INT    NOT NULL CHECK (quantity > 0),
  unit_price DECIMAL(12, 2) NOT NULL,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id)   REFERENCES orders(id)   ON DELETE CASCADE,
  FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE RESTRICT
);
```

### AUTO_INCREMENT Notes

- Generates sequential integers: 1, 2, 3, ...
- **Gaps are normal** — if an insert is rolled back, the AUTO_INCREMENT counter still advances.
- Do **not** use auto-increment values as business-visible sequence numbers if gapless order is required.

---

## Inspecting Table Structure

```sql
-- Show columns, types, and basic constraints
DESCRIBE customers;

-- Show the full CREATE TABLE statement (including indexes)
SHOW CREATE TABLE customers;

-- List all tables in the active database
SHOW TABLES;

-- List tables matching a pattern
SHOW TABLES LIKE 'order%';
```

---

## Modifying Tables

### Add a Column

```sql
ALTER TABLE customers
  ADD COLUMN birth_date DATE AFTER phone;
```

### Modify a Column (Change Type or Constraint)

```sql
ALTER TABLE customers
  MODIFY COLUMN phone VARCHAR(30);
```

### Rename a Column (MySQL 8+)

```sql
ALTER TABLE customers
  RENAME COLUMN phone TO phone_number;
```

### Drop a Column

```sql
ALTER TABLE customers
  DROP COLUMN birth_date;
```

### Rename a Table

```sql
RENAME TABLE customers TO clients;
-- or
ALTER TABLE customers RENAME TO clients;
```

### Add a New Column with a Default

```sql
ALTER TABLE products
  ADD COLUMN stock INT NOT NULL DEFAULT 0;
```

---

## Removing Tables

### DROP TABLE — Removes the table completely

```sql
DROP TABLE IF EXISTS order_products;
```

### TRUNCATE TABLE — Removes all rows but keeps the structure

```sql
TRUNCATE TABLE audit_logs;
```

### DROP vs TRUNCATE vs DELETE

| Action | Removes | Keeps Structure | Transactional | Resets AUTO_INCREMENT |
|--------|---------|-----------------|---------------|----------------------|
| `DROP TABLE` | Table + all data | ❌ | ❌ | N/A |
| `TRUNCATE TABLE` | All rows | ✅ | ❌ (DDL) | ✅ Yes |
| `DELETE FROM` | Selected rows | ✅ | ✅ | ❌ No |

---

## CRUD Operations

### Create — INSERT

#### Single Row

```sql
INSERT INTO customers (full_name, email)
VALUES ('Ana Putri', 'ana@example.com');
```

#### Multiple Rows (Efficient Bulk Insert)

```sql
INSERT INTO customers (full_name, email)
VALUES
  ('Budi Santoso', 'budi@example.com'),
  ('Citra Dewi',   'citra@mail.com'),
  ('Dian Pratama', 'dian@example.com');
```

#### Upsert (Insert or Update on Duplicate Key)

```sql
INSERT INTO products (sku, name, price, stock)
VALUES ('SKU-001', 'Laptop Stand', 250000.00, 50)
ON DUPLICATE KEY UPDATE
  price = VALUES(price),
  stock = stock + VALUES(stock);
```

### Read — SELECT

#### Basic Select

```sql
SELECT id, full_name, email
FROM customers;
```

#### Select with Filter

```sql
SELECT id, full_name
FROM customers
WHERE is_active = 1
  AND email LIKE '%@example.com';
```

#### Count Rows

```sql
SELECT COUNT(*) AS total_customers
FROM customers;
```

### Update — UPDATE

```sql
-- Always include WHERE to avoid updating all rows
UPDATE customers
SET full_name = 'Ana P.', phone = '08123456789'
WHERE id = 1;
```

#### Safe Update Pattern

```sql
-- Step 1: Preview what will be updated
SELECT id, full_name FROM customers WHERE id = 1;

-- Step 2: If correct, run the update
UPDATE customers
SET full_name = 'Ana P.'
WHERE id = 1;
```

### Delete — DELETE

```sql
-- Delete a specific row
DELETE FROM customers
WHERE id = 1;

-- Delete multiple rows matching a condition
DELETE FROM customers
WHERE is_active = 0
  AND created_at < '2025-01-01';
```

> **Warning**: `DELETE FROM customers` without a `WHERE` clause deletes **all rows**.

---

## Safety Checklist for UPDATE and DELETE

- ✅ Run a `SELECT` with the same `WHERE` condition first to preview affected rows.
- ✅ In production, wrap inside a transaction so you can `ROLLBACK` if something is wrong.
- ✅ Check the row count before committing: `SELECT ROW_COUNT();` returns affected rows from the last DML.
- ✅ Use strict `WHERE` filters — avoid patterns like `WHERE status != 'active'` that can grow unexpectedly.

```sql
START TRANSACTION;
DELETE FROM orders WHERE customer_id = 5 AND status = 'cancelled';
SELECT ROW_COUNT();  -- Verify: how many rows were deleted?
-- If correct:
COMMIT;
-- If wrong:
-- ROLLBACK;
```

---

## Best Practices

- Always specify column names in `INSERT` — do not rely on column order.
- Avoid `SELECT *` in application queries — select only the columns you need.
- Add `NOT NULL` and `DEFAULT` constraints to columns wherever sensible.
- Index columns frequently used in `WHERE`, `JOIN`, and `ORDER BY` clauses.
- Use transactions for multi-step modifications.

## Common Mistakes

- Running `UPDATE` or `DELETE` without a `WHERE` clause.
- Relying on implicit column order in `INSERT INTO table VALUES (...)`.
- Using `TRUNCATE` when you need the operation inside a transaction.
- Forgetting to run `DESCRIBE` or `SHOW CREATE TABLE` before writing queries on an unfamiliar table.

## Next Step

Continue to [8-filtering-sorting-pagination.md](8-filtering-sorting-pagination.md).


