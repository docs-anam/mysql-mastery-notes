# Subqueries

## Definition

A subquery is a query nested inside another SQL query.

## Example in WHERE

```sql
SELECT id, full_name
FROM customers
WHERE id IN (
  SELECT customer_id
  FROM orders
  WHERE total_amount > 1000000
);
```

## Next Step

Continue to [40-set-operator.md](40-set-operator.md).

