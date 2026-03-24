# Alias in SQL

## Column Alias

```sql
SELECT
  full_name AS customer_name,
  created_at AS registered_at
FROM customers;
```

## Table Alias

```sql
SELECT c.id, c.full_name
FROM customers AS c;
```

## Why Use Alias

- Improve readability
- Shorten long table names
- Clarify computed columns

## Next Step

Continue to [20-where-operator.md](20-where-operator.md).
