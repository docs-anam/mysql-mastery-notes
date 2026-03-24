# Join Types

## Common Join Types

- `INNER JOIN`: matched rows only
- `LEFT JOIN`: all rows from the left table plus matches from the right table
- `RIGHT JOIN`: all rows from the right table plus matches from the left table
- `CROSS JOIN`: Cartesian product

## Example

```sql
SELECT c.full_name, o.id AS order_id
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;
```

## Next Step

Continue to [39-sub-query.md](39-sub-query.md).
