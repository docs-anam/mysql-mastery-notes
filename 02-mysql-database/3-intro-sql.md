# Introduction to SQL

## What is SQL?

**SQL (Structured Query Language)** is the standard language for communicating with relational databases. It lets you define structure, manipulate data, retrieve information, and control access — all through a declarative syntax.

> SQL is **declarative**: you describe *what* you want, not *how* to get it. The MySQL query optimizer figures out the most efficient execution plan.

## SQL Categories

SQL is organized into five categories based on what each group of statements does:

```
SQL
├── DDL  →  Define and modify structure (tables, indexes)
├── DML  →  Manipulate row-level data (insert, update, delete)
├── DQL  →  Query and retrieve data (select)
├── TCL  →  Manage transactions (commit, rollback)
└── DCL  →  Control user permissions (grant, revoke)
```

---

### 1. DDL — Data Definition Language

Defines and modifies the **structure** of database objects.

| Statement | Purpose |
|-----------|---------|
| `CREATE` | Create a database, table, index, or view |
| `ALTER` | Modify an existing table's structure |
| `DROP` | Remove a database, table, or index permanently |
| `TRUNCATE` | Remove all rows from a table (fast, resets AUTO_INCREMENT) |
| `RENAME` | Rename a table |

```sql
-- Create a table
CREATE TABLE products (
  id    BIGINT       PRIMARY KEY AUTO_INCREMENT,
  name  VARCHAR(120) NOT NULL,
  price DECIMAL(12, 2) NOT NULL
);

-- Add a column
ALTER TABLE products ADD COLUMN stock INT NOT NULL DEFAULT 0;

-- Remove the table permanently
DROP TABLE IF EXISTS products;

-- Remove all rows but keep the table structure
TRUNCATE TABLE products;
```

---

### 2. DML — Data Manipulation Language

Modifies the **data** (rows) inside tables.

| Statement | Purpose |
|-----------|---------|
| `INSERT` | Add new rows |
| `UPDATE` | Modify existing rows |
| `DELETE` | Remove rows |

```sql
-- Insert a row
INSERT INTO products (name, price, stock)
VALUES ('Laptop Stand', 250000.00, 50);

-- Update a row
UPDATE products
SET price = 275000.00
WHERE id = 1;

-- Delete a row
DELETE FROM products
WHERE id = 1;
```

> **Safety Rule**: Always include a `WHERE` clause with `UPDATE` and `DELETE`. Without it, every row in the table is affected.

---

### 3. DQL — Data Query Language

Retrieves data from one or more tables.

| Statement | Purpose |
|-----------|---------|
| `SELECT` | Read and return rows from tables |

```sql
-- Basic query
SELECT id, name, price
FROM products
WHERE stock > 0
ORDER BY price ASC
LIMIT 10;

-- Query across joined tables
SELECT c.full_name, o.order_date, o.total_amount
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.total_amount > 500000
ORDER BY o.order_date DESC;
```

---

### 4. TCL — Transaction Control Language

Controls the **boundaries of transactions**.

| Statement | Purpose |
|-----------|---------|
| `START TRANSACTION` | Begin a transaction block |
| `COMMIT` | Permanently save all changes made in the transaction |
| `ROLLBACK` | Undo all changes since the last `COMMIT` |
| `SAVEPOINT name` | Mark a named checkpoint within a transaction |
| `ROLLBACK TO name` | Undo to a specific savepoint |

```sql
START TRANSACTION;
  UPDATE accounts SET balance = balance - 100000 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100000 WHERE id = 2;
COMMIT;  -- Both updates are saved atomically
```

```sql
START TRANSACTION;
  DELETE FROM orders WHERE id = 5;
ROLLBACK;  -- Delete is undone — order 5 is still there
```

---

### 5. DCL — Data Control Language

Manages **user permissions** and access rights.

| Statement | Purpose |
|-----------|---------|
| `GRANT` | Give a user one or more privileges |
| `REVOKE` | Remove one or more privileges from a user |

```sql
-- Give app_user read/write access to the bookstore database
GRANT SELECT, INSERT, UPDATE, DELETE ON bookstore.* TO 'app_user'@'%';

-- Remove DELETE permission
REVOKE DELETE ON bookstore.* FROM 'app_user'@'%';
```

---

## SQL Query Execution Order

SQL is written in one order but **evaluated in a different order**. Understanding this prevents common mistakes like referencing an alias in the wrong clause.

```
Written Order         Execution Order
─────────────         ───────────────
SELECT            →   1.  FROM
FROM              →   2.  JOIN / ON
JOIN              →   3.  WHERE
WHERE             →   4.  GROUP BY
GROUP BY          →   5.  HAVING
HAVING            →   6.  SELECT
ORDER BY          →   7.  DISTINCT
LIMIT             →   8.  ORDER BY
                  →   9.  LIMIT / OFFSET
```

> **Why this matters**: You **cannot** use a `SELECT` alias in a `WHERE` clause — `WHERE` runs before `SELECT`. You **can** use it in `ORDER BY` because `ORDER BY` runs after `SELECT`.

## Complete Query Anatomy

```sql
SELECT                            -- 6. Choose which columns to return
  c.full_name,                    --    Alias 'c' for customers table
  COUNT(o.id)  AS total_orders,   --    Aggregate function
  SUM(o.total) AS total_spent     --    Another aggregate
FROM customers c                  -- 1. Start with the customers table
JOIN orders o                     -- 2. Join the orders table
  ON o.customer_id = c.id         --    Join condition
WHERE c.is_active = 1             -- 3. Filter: only active customers
GROUP BY c.id, c.full_name        -- 4. Group rows by customer
HAVING COUNT(o.id) >= 3           -- 5. Filter groups: 3 or more orders
ORDER BY total_spent DESC         -- 8. Sort by total spent (uses alias)
LIMIT 10;                         -- 9. Return only the top 10
```

## SQL Syntax Rules

- Keywords are **case-insensitive**: `SELECT`, `select`, and `Select` are equivalent
- **Convention**: write SQL keywords in `UPPERCASE`, identifiers in `lowercase`
- Statements end with a **semicolon** `;`
- String values use **single quotes**: `'hello'` (not double quotes)
- Identifiers with spaces or reserved words are wrapped in **backticks**: `` `order` ``

```sql
-- Comments in SQL
-- This is a single-line comment

/*
  This is a
  multi-line comment
*/
```

## NULL in SQL

`NULL` means "no value" or "unknown". It behaves differently from zero or an empty string.

```sql
-- WRONG: = is never true for NULL
SELECT * FROM customers WHERE phone = NULL;

-- CORRECT: use IS NULL
SELECT * FROM customers WHERE phone IS NULL;

-- CORRECT: use IS NOT NULL
SELECT * FROM customers WHERE phone IS NOT NULL;
```

## Best Practices

- Specify column names explicitly in `SELECT` — avoid `SELECT *` in production code.
- Always use a `WHERE` clause with `UPDATE` and `DELETE`.
- Test DML statements as `SELECT` first to verify the affected rows.
- Keep DDL changes in version-controlled migration scripts.
- Use meaningful aliases for joined tables and computed columns.

## Common Mistakes

- Running `DELETE FROM table` without a `WHERE` clause — wipes all rows.
- Using `=` to check for `NULL` — returns no results; use `IS NULL`.
- Referencing a `SELECT` alias in a `WHERE` clause — fails because execution order.
- Confusing `TRUNCATE` (DDL, not transactional) with `DELETE` (DML, transactional).

## Next Step

Continue to [4-install-mysql.md](4-install-mysql.md).

