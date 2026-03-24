# Table Relationships, Joins, and Advanced Queries

## Relationship Types

In a relational database, tables are connected through **foreign keys**. There are three fundamental relationship types.

---

## 1. One-to-Many (1:M)

**The most common relationship.** One row in table A is related to many rows in table B.

```
CUSTOMERS                      ORDERS
┌─────────────────┐            ┌──────────────────────┐
│ id (PK)         │            │ id (PK)              │
│ full_name       ├────[1]─[M]─┤ customer_id (FK) ───▶│ (references customers.id)
│ email           │            │ order_date           │
└─────────────────┘            │ total_amount         │
                               └──────────────────────┘
One customer → many orders
```

```sql
CREATE TABLE customers (
  id        BIGINT       PRIMARY KEY AUTO_INCREMENT,
  full_name VARCHAR(120) NOT NULL
);

CREATE TABLE orders (
  id          BIGINT    PRIMARY KEY AUTO_INCREMENT,
  customer_id BIGINT    NOT NULL,
  order_date  DATE      NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE CASCADE
);
```

---

## 2. One-to-One (1:1)

One row in table A is related to exactly one row in table B. Used to split rarely-used columns into a separate table.

```
USERS                     USER_PROFILES
┌──────────────┐           ┌──────────────────────┐
│ id (PK)      ├────[1:1]──┤ user_id (PK, FK) ───▶│ (references users.id)
│ email        │           │ full_name             │
│ password     │           │ avatar_url            │
└──────────────┘           │ bio                   │
                           └──────────────────────┘
One user → one profile
```

```sql
CREATE TABLE users (
  id       BIGINT       PRIMARY KEY AUTO_INCREMENT,
  email    VARCHAR(120) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL
);

CREATE TABLE user_profiles (
  user_id    BIGINT       PRIMARY KEY,         -- PK is also the FK
  full_name  VARCHAR(120) NOT NULL,
  avatar_url VARCHAR(255),
  bio        TEXT,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

---

## 3. Many-to-Many (M:N)

Requires a **junction table** (also called a bridge or pivot table) that holds foreign keys to both sides.

```
STUDENTS                 STUDENT_COURSES        COURSES
┌────────────┐           ┌───────────────┐       ┌────────────┐
│ id (PK)    ├────[1:M]──┤ student_id FK │       │ id (PK)    │
│ name       │           │ course_id  FK ├[M:1]──┤ name       │
└────────────┘           │ enrolled_at   │       │ credits    │
                         └───────────────┘       └────────────┘
Many students ↔ many courses (through junction table)
```

```sql
CREATE TABLE students (
  id   BIGINT       PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(120) NOT NULL
);

CREATE TABLE courses (
  id      BIGINT       PRIMARY KEY AUTO_INCREMENT,
  name    VARCHAR(120) NOT NULL,
  credits INT          NOT NULL DEFAULT 3
);

CREATE TABLE student_courses (
  student_id  BIGINT    NOT NULL,
  course_id   BIGINT    NOT NULL,
  enrolled_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  grade       CHAR(2),
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (course_id)  REFERENCES courses(id)  ON DELETE CASCADE
);
```

---

## JOIN Types

A `JOIN` combines rows from two or more tables based on a related column.

```
Table A         Table B        INNER JOIN     LEFT JOIN      RIGHT JOIN
┌───┐           ┌───┐
│ A │           │ B │           ┌────┐         ┌────────┐     ┌──────────┐
│   │           │   │           │A∩B │         │ A + A∩B│     │ B + A∩B  │
│   │           │   │           └────┘         └────────┘     └──────────┘
└───┘           └───┘        Only matching   All of A +      All of B +
                              rows in both    matching B      matching A
```

### INNER JOIN

Returns only rows where the join condition matches in **both** tables.

```sql
SELECT c.full_name, o.order_date, o.total_amount
FROM customers c
INNER JOIN orders o ON o.customer_id = c.id;
-- Returns only customers who HAVE orders
-- Customers with no orders are excluded
```

### LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left table**, with matching rows from the right table. Unmatched right-side columns are `NULL`.

```sql
SELECT c.full_name, o.order_date, o.total_amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;
-- Returns ALL customers
-- Customers with no orders appear with NULL in order columns

-- Find customers who have NEVER placed an order
SELECT c.id, c.full_name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.id IS NULL;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right table**, with matching rows from the left table. Less commonly used — most RIGHT JOINs can be rewritten as a LEFT JOIN by swapping table order.

```sql
SELECT c.full_name, o.order_date
FROM customers c
RIGHT JOIN orders o ON o.customer_id = c.id;
-- Returns ALL orders, even if customer was deleted
```

### CROSS JOIN

Returns every combination of rows — the **Cartesian product**. Produces `rows_A × rows_B` rows.

```sql
-- Generate all combinations of sizes and colors
SELECT s.size, c.color
FROM sizes s
CROSS JOIN colors c;
-- 5 sizes × 4 colors = 20 combinations
```

### INNER JOIN vs LEFT JOIN

| Scenario | Use |
|---------|-----|
| You want only matching rows from both tables | `INNER JOIN` |
| You want all rows from the left table, even with no match | `LEFT JOIN` |
| You want to find rows with NO match (orphan detection) | `LEFT JOIN ... WHERE right.id IS NULL` |

---

## Multi-Table JOINs

```sql
-- Order details with customer and product info
SELECT
  c.full_name         AS customer,
  o.order_date,
  p.name              AS product,
  oi.quantity,
  oi.unit_price,
  (oi.quantity * oi.unit_price) AS line_total
FROM orders o
JOIN customers  c  ON c.id  = o.customer_id
JOIN order_items oi ON oi.order_id = o.id
JOIN products   p  ON p.id  = oi.product_id
WHERE o.status = 'completed'
ORDER BY o.order_date DESC;
```

---

## Subqueries

A **subquery** is a `SELECT` statement nested inside another SQL statement.

### Subquery in WHERE (IN)

```sql
-- Find customers who have placed at least one high-value order
SELECT id, full_name
FROM customers
WHERE id IN (
  SELECT DISTINCT customer_id
  FROM orders
  WHERE total_amount > 1000000
);
```

### Correlated Subquery

A subquery that references columns from the outer query. Evaluated once per row.

```sql
-- For each customer, show their most recent order date
SELECT
  c.full_name,
  (SELECT MAX(order_date) FROM orders WHERE customer_id = c.id) AS last_order
FROM customers c;
```

### EXISTS vs IN

```sql
-- EXISTS: returns true if subquery returns any row (stops at first match)
SELECT c.full_name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders WHERE customer_id = c.id AND status = 'completed'
);

-- Both EXISTS and IN work here, but EXISTS is often faster
-- when the subquery would return many rows
```

| | `IN` | `EXISTS` |
|-|------|---------|
| Evaluates | Entire subquery first | Row-by-row; stops at first match |
| Best when | Subquery result is small | Subquery result is large |
| NULL behavior | `IN` can produce unexpected results with NULLs | `EXISTS` is safer |

---

## Set Operators

### UNION vs UNION ALL

```sql
-- UNION: combines results and removes duplicates
SELECT email FROM customers_2025
UNION
SELECT email FROM customers_2026;

-- UNION ALL: combines results, keeping all rows including duplicates (faster)
SELECT email FROM customers_2025
UNION ALL
SELECT email FROM customers_2026;
```

| | `UNION` | `UNION ALL` |
|-|---------|------------|
| Duplicates | Removed (slower) | Kept (faster) |
| Use when | You need distinct rows | Duplicates are impossible or acceptable |

> **Both queries must have the same number of columns and compatible types.**

---

## Practical Example: Customer Order Summary

```sql
SELECT
  c.id,
  c.full_name,
  COUNT(o.id)           AS total_orders,
  COALESCE(SUM(o.total_amount), 0) AS total_spent,
  MAX(o.order_date)     AS last_order_date
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id AND o.status = 'completed'
GROUP BY c.id, c.full_name
ORDER BY total_spent DESC
LIMIT 10;
```

---

## Best Practices

- Define relationship **cardinality** explicitly before writing queries.
- Always use **explicit join conditions** — never rely on implicit Cartesian joins.
- Use **table aliases** consistently (`customers c`, `orders o`) for readability.
- Validate row counts when joining multiple tables — unexpected duplicates are a common source of bugs.
- Prefer `LEFT JOIN ... WHERE right.id IS NULL` to find orphaned or unmatched records.

## Common Mistakes

- Missing a `JOIN` condition — causes an accidental Cartesian product (millions of rows).
- Using `INNER JOIN` when some parent rows have no children and you need to include them.
- Ignoring the duplicate-removal difference between `UNION` and `UNION ALL`.
- Using correlated subqueries on large tables without checking performance with `EXPLAIN`.

## Next Step

Continue to [12-transaction.md](12-transaction.md).

