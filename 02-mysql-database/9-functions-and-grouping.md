# SQL Functions and Grouping

## Overview

MySQL provides many **built-in functions** to transform, format, and compute values directly inside queries. This chapter covers numeric, string, date/time, and flow-control functions, followed by aggregation and grouping.

---

## 1. Numeric Functions

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `ABS(x)` | Absolute value | `ABS(-15)` | `15` |
| `ROUND(x, d)` | Round to `d` decimal places | `ROUND(3.456, 2)` | `3.46` |
| `CEIL(x)` | Smallest integer ≥ x | `CEIL(3.2)` | `4` |
| `FLOOR(x)` | Largest integer ≤ x | `FLOOR(3.9)` | `3` |
| `MOD(a, b)` | Remainder of a ÷ b | `MOD(17, 5)` | `2` |
| `POW(x, y)` | x raised to power y | `POW(2, 10)` | `1024` |
| `SQRT(x)` | Square root | `SQRT(144)` | `12` |
| `TRUNCATE(x, d)` | Truncate to `d` decimal places (no rounding) | `TRUNCATE(3.789, 2)` | `3.78` |

```sql
-- Practical: round product prices
SELECT
  name,
  price,
  ROUND(price * 1.11, 0) AS price_with_tax
FROM products;
```

---

## 2. String Functions

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `CONCAT(a, b, ...)` | Join strings together | `CONCAT('Hello', ' ', 'World')` | `'Hello World'` |
| `CONCAT_WS(sep, ...)` | Join with separator | `CONCAT_WS(', ', 'Jakarta', 'Indonesia')` | `'Jakarta, Indonesia'` |
| `UPPER(s)` | Convert to uppercase | `UPPER('hello')` | `'HELLO'` |
| `LOWER(s)` | Convert to lowercase | `LOWER('HELLO')` | `'hello'` |
| `LENGTH(s)` | Length in bytes | `LENGTH('café')` | `5` |
| `CHAR_LENGTH(s)` | Length in characters | `CHAR_LENGTH('café')` | `4` |
| `TRIM(s)` | Remove leading/trailing spaces | `TRIM('  hi  ')` | `'hi'` |
| `LTRIM(s)` / `RTRIM(s)` | Remove spaces from left/right | `LTRIM('  hi')` | `'hi'` |
| `SUBSTRING(s, pos, len)` | Extract part of a string | `SUBSTRING('MySQL', 3, 3)` | `'SQL'` |
| `REPLACE(s, from, to)` | Replace occurrences | `REPLACE('abc', 'b', 'X')` | `'aXc'` |
| `LEFT(s, n)` | First `n` characters | `LEFT('Hello', 3)` | `'Hel'` |
| `RIGHT(s, n)` | Last `n` characters | `RIGHT('Hello', 3)` | `'llo'` |
| `INSTR(s, sub)` | Position of first occurrence | `INSTR('Hello', 'l')` | `3` |

```sql
-- Practical: format customer display name
SELECT
  CONCAT(first_name, ' ', last_name) AS full_name,
  LOWER(email)                       AS email,
  CHAR_LENGTH(bio)                   AS bio_length
FROM customers;
```

---

## 3. Date and Time Functions

| Function | Description | Example Result |
|----------|-------------|----------------|
| `NOW()` | Current date and time | `2026-03-24 10:35:00` |
| `CURDATE()` | Current date only | `2026-03-24` |
| `CURTIME()` | Current time only | `10:35:00` |
| `DATE(dt)` | Extract date from datetime | `DATE('2026-03-24 10:30:00')` → `2026-03-24` |
| `TIME(dt)` | Extract time from datetime | `TIME('2026-03-24 10:30:00')` → `10:30:00` |
| `YEAR(d)` | Extract year | `YEAR('2026-03-24')` → `2026` |
| `MONTH(d)` | Extract month | `MONTH('2026-03-24')` → `3` |
| `DAY(d)` | Extract day | `DAY('2026-03-24')` → `24` |
| `DATEDIFF(d1, d2)` | Days between two dates | `DATEDIFF('2026-03-24', '2026-01-01')` → `82` |
| `DATE_ADD(dt, INTERVAL n unit)` | Add an interval | `DATE_ADD('2026-01-01', INTERVAL 30 DAY)` → `2026-01-31` |
| `DATE_SUB(dt, INTERVAL n unit)` | Subtract an interval | `DATE_SUB(NOW(), INTERVAL 7 DAY)` |
| `DATE_FORMAT(dt, fmt)` | Format date as string | `DATE_FORMAT(NOW(), '%d %M %Y')` → `'24 March 2026'` |

```sql
-- Orders placed in the last 30 days
SELECT id, order_date, total_amount
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
ORDER BY order_date DESC;

-- Format dates for display
SELECT
  id,
  DATE_FORMAT(created_at, '%d %b %Y') AS formatted_date
FROM orders;
```

### Common Date Format Codes

| Code | Meaning | Example |
|------|---------|---------|
| `%Y` | 4-digit year | `2026` |
| `%m` | 2-digit month | `03` |
| `%d` | 2-digit day | `24` |
| `%H` | 2-digit hour (24h) | `10` |
| `%i` | 2-digit minute | `35` |
| `%s` | 2-digit second | `00` |
| `%b` | Abbreviated month name | `Mar` |

---

## 4. Flow Control Functions

### CASE Expression

`CASE` evaluates conditions and returns a value — like an if/else inside a query.

```sql
-- Categorize orders by amount
SELECT
  id,
  total_amount,
  CASE
    WHEN total_amount >= 1000000 THEN 'high'
    WHEN total_amount >= 300000  THEN 'medium'
    ELSE                              'low'
  END AS order_segment
FROM orders;
```

### IF Function

```sql
-- IF(condition, value_if_true, value_if_false)
SELECT
  id,
  full_name,
  IF(is_active = 1, 'Active', 'Inactive') AS status_label
FROM customers;
```

### IFNULL and COALESCE

```sql
-- IFNULL: return alternative if value is NULL
SELECT full_name, IFNULL(phone, 'N/A') AS phone
FROM customers;

-- COALESCE: return first non-NULL from the list
SELECT full_name, COALESCE(mobile, phone, email, 'No contact') AS contact
FROM customers;
```

---

## 5. Aggregate Functions

Aggregate functions compute a **single value from a group of rows**.

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Count all rows in the group |
| `COUNT(col)` | Count non-NULL values in a column |
| `SUM(col)` | Sum of all values |
| `AVG(col)` | Average of all values |
| `MIN(col)` | Minimum value |
| `MAX(col)` | Maximum value |
| `GROUP_CONCAT(col)` | Concatenate values from all rows in the group |

```sql
-- Sales summary per customer
SELECT
  customer_id,
  COUNT(*)                         AS total_orders,
  SUM(total_amount)                AS total_spent,
  AVG(total_amount)                AS avg_order_value,
  MIN(order_date)                  AS first_order,
  MAX(order_date)                  AS last_order
FROM orders
GROUP BY customer_id;
```

---

## 6. GROUP BY and HAVING

### GROUP BY

Collapses multiple rows into one row per group, enabling aggregate calculations.

```sql
-- Monthly revenue report
SELECT
  YEAR(order_date)  AS year,
  MONTH(order_date) AS month,
  COUNT(*)          AS total_orders,
  SUM(total_amount) AS monthly_revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year DESC, month DESC;
```

### HAVING

Filters **groups** (applied after `GROUP BY`). Use `WHERE` to filter individual rows and `HAVING` to filter groups.

```sql
-- Find customers who have placed 5 or more orders
SELECT
  customer_id,
  COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 5
ORDER BY total_orders DESC;
```

### WHERE vs HAVING

| | `WHERE` | `HAVING` |
|-|---------|---------|
| Filters | Individual rows | Groups (after GROUP BY) |
| Can use aggregates | ❌ No | ✅ Yes |
| Runs | Before GROUP BY | After GROUP BY |
| Performance | More efficient (filters early) | Less efficient (filters late) |

```sql
-- WRONG: cannot use aggregate in WHERE
SELECT customer_id, COUNT(*) FROM orders
WHERE COUNT(*) > 5           -- ❌ Error!
GROUP BY customer_id;

-- CORRECT: use HAVING for aggregate filter
SELECT customer_id, COUNT(*) FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;         -- ✅
```

---

## Practical Example: Sales Report

```sql
SELECT
  c.full_name                             AS customer,
  COUNT(o.id)                             AS total_orders,
  SUM(o.total_amount)                     AS total_spent,
  ROUND(AVG(o.total_amount), 0)           AS avg_order,
  DATE_FORMAT(MAX(o.order_date), '%d %b %Y') AS last_order_date,
  CASE
    WHEN SUM(o.total_amount) >= 10000000 THEN 'Platinum'
    WHEN SUM(o.total_amount) >= 5000000  THEN 'Gold'
    ELSE 'Regular'
  END                                     AS tier
FROM customers c
JOIN orders o ON o.customer_id = c.id
WHERE o.status = 'completed'
  AND o.order_date >= '2026-01-01'
GROUP BY c.id, c.full_name
HAVING COUNT(o.id) >= 2
ORDER BY total_spent DESC
LIMIT 20;
```

---

## Best Practices

- Keep expressions explicit and alias all computed columns clearly.
- Use `WHERE` to filter rows before grouping, and `HAVING` only for aggregate-based group filters.
- Validate grouping logic with small datasets before running on production.
- When using `GROUP BY`, every column in `SELECT` must either be in `GROUP BY` or wrapped in an aggregate function.

## Common Mistakes

- Putting non-aggregated, non-grouped columns in `SELECT` with `GROUP BY` — produces unpredictable results.
- Using `HAVING` where `WHERE` would be faster — `WHERE` eliminates rows earlier in processing.
- Using string functions on large, unindexed `TEXT` columns without performance checks.
- Treating `DATE()` extraction as zero-cost on large tables — it prevents index usage.

## Next Step

Continue to [10-constraint.md](10-constraint.md) to learn about data integrity constraints, indexes, and full-text search.

