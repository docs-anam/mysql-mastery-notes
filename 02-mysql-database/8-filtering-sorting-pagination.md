# Filtering, Sorting, and Pagination

## Aliases

**Aliases** give a temporary name to a column or table within a query. They exist only for the duration of that query.

```sql
-- Column alias
SELECT full_name AS name, created_at AS registered_at
FROM customers;

-- Table alias (shortens join syntax)
SELECT c.full_name, o.total_amount
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.id;
```

> Table aliases are especially valuable in multi-join queries — they make the intent clear and prevent ambiguous column references.

---

## Filtering with WHERE

The `WHERE` clause filters rows **before** any grouping or aggregation.

```sql
SELECT id, full_name, email
FROM customers
WHERE is_active = 1
  AND created_at >= '2026-01-01';
```

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal | `status = 'active'` |
| `!=` or `<>` | Not equal | `status != 'cancelled'` |
| `>` | Greater than | `price > 100000` |
| `<` | Less than | `price < 500000` |
| `>=` | Greater than or equal | `age >= 18` |
| `<=` | Less than or equal | `stock <= 10` |

### Range: BETWEEN

```sql
-- Inclusive range: price from 100,000 to 500,000
SELECT id, name, price
FROM products
WHERE price BETWEEN 100000 AND 500000;

-- Date range
SELECT id, order_date
FROM orders
WHERE order_date BETWEEN '2026-01-01' AND '2026-03-31';
```

### Set Membership: IN

```sql
-- Match any value in the list
SELECT id, full_name
FROM customers
WHERE city IN ('Jakarta', 'Bandung', 'Surabaya');

-- Exclude a set
SELECT id, name
FROM products
WHERE status NOT IN ('discontinued', 'draft');
```

### Pattern Matching: LIKE

| Pattern | Matches |
|---------|---------|
| `'A%'` | Starts with A |
| `'%com'` | Ends with "com" |
| `'%gmail%'` | Contains "gmail" anywhere |
| `'_udi'` | Any single char followed by "udi" (e.g., "Budi") |

```sql
-- Find all Gmail customers
SELECT id, full_name, email
FROM customers
WHERE email LIKE '%@gmail.com';

-- Find customers whose name starts with 'A'
SELECT id, full_name
FROM customers
WHERE full_name LIKE 'A%';
```

### NULL Handling: IS NULL / IS NOT NULL

```sql
-- Find customers with no phone number
SELECT id, full_name
FROM customers
WHERE phone IS NULL;

-- Find customers who have provided a phone number
SELECT id, full_name, phone
FROM customers
WHERE phone IS NOT NULL;
```

> **Important**: `WHERE phone = NULL` never returns rows. Always use `IS NULL`.

### Combining Conditions: AND / OR / NOT

```sql
-- AND: both conditions must be true
SELECT * FROM orders
WHERE status = 'pending'
  AND total_amount > 500000;

-- OR: either condition is true
SELECT * FROM customers
WHERE city = 'Jakarta'
   OR city = 'Bogor';

-- NOT: negates a condition
SELECT * FROM products
WHERE NOT status = 'discontinued';

-- Complex combination (use parentheses to be explicit)
SELECT * FROM orders
WHERE (status = 'pending' OR status = 'processing')
  AND total_amount >= 200000
  AND created_at >= '2026-01-01';
```

---

## Sorting with ORDER BY

```sql
-- Single column, ascending (default)
SELECT id, full_name, created_at
FROM customers
ORDER BY created_at;

-- Multiple columns: primary sort DESC, secondary sort ASC
SELECT id, full_name, city, created_at
FROM customers
ORDER BY city ASC, created_at DESC;
```

- `ASC` — ascending (A→Z, 0→9), **default if omitted**
- `DESC` — descending (Z→A, 9→0)

> For **paginated queries**, always use a deterministic `ORDER BY` (e.g., `ORDER BY id`) — without it, the same `LIMIT/OFFSET` may return different rows on successive calls.

---

## Pagination with LIMIT and OFFSET

```sql
-- First page: rows 1–10
SELECT id, full_name
FROM customers
ORDER BY id
LIMIT 10 OFFSET 0;

-- Second page: rows 11–20
SELECT id, full_name
FROM customers
ORDER BY id
LIMIT 10 OFFSET 10;

-- Third page: rows 21–30
SELECT id, full_name
FROM customers
ORDER BY id
LIMIT 10 OFFSET 20;
```

### Shorthand Syntax

```sql
-- LIMIT [offset,] row_count
SELECT id, full_name FROM customers ORDER BY id LIMIT 20, 10;
-- Equivalent to LIMIT 10 OFFSET 20
```

### Counting Total Pages

```sql
-- Get total row count first
SELECT COUNT(*) AS total FROM customers;

-- Then use: total_pages = CEIL(total / page_size)
```

---

## Removing Duplicates with DISTINCT

```sql
-- All unique cities from the customers table
SELECT DISTINCT city FROM customers;

-- Unique combinations of city + country
SELECT DISTINCT city, country FROM customers;
```

> Overusing `DISTINCT` to hide duplicate rows often means there's a join producing unintended duplicates — investigate the root cause rather than masking it.

---

## Practical Example: Product Search Page

```sql
-- User searches "laptop", filters by price range, sorted by price, page 1
SELECT
  p.id,
  p.name,
  p.price,
  p.stock
FROM products p
WHERE p.status = 'active'
  AND p.name LIKE '%laptop%'
  AND p.price BETWEEN 2000000 AND 15000000
  AND p.stock > 0
ORDER BY p.price ASC
LIMIT 12 OFFSET 0;
```

---

## Best Practices

- Combine `WHERE`, `ORDER BY`, and `LIMIT` intentionally — each affects correctness and performance.
- Always use `IS NULL` / `IS NOT NULL` for null checks, never `= NULL`.
- Use `BETWEEN` for range filters — it is inclusive on both ends and clearly communicates intent.
- Always use `ORDER BY` when using `LIMIT` / `OFFSET` to guarantee consistent pagination.
- Keep aliases meaningful — `c` for `customers`, `o` for `orders`, not arbitrary letters.

## Common Mistakes

- Using `= NULL` instead of `IS NULL` — returns zero rows silently.
- Using `LIMIT/OFFSET` without a stable `ORDER BY` — different pages may overlap or skip rows.
- Applying `LIKE '%keyword%'` on large unindexed columns without performance awareness — causes full table scans.
- Using `OR` without parentheses in complex conditions — operator precedence may produce unexpected results.

## Next Step

Continue to [9-functions-and-grouping.md](9-functions-and-grouping.md).

