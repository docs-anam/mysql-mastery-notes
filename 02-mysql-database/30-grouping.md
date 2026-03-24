# Grouping Data with GROUP BY

## Basic Grouping

```sql
SELECT customer_id, COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id;
```

## HAVING Clause

```sql
SELECT customer_id, COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 5;
```

## Next Step

Continue to [31-constraint.md](31-constraint.md).

