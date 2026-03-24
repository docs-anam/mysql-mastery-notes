# JOIN Basics

## Purpose

`JOIN` combines rows from two or more tables based on related columns.

## Inner Join Example

```sql
SELECT o.id, c.full_name, o.total_amount
FROM orders o
JOIN customers c ON c.id = o.customer_id;
```

## Notes

- Always qualify columns with table aliases.
- Use explicit `JOIN ... ON ...` syntax for clarity.

## Next Step

Continue to [35-one-to-one-relationship.md](35-one-to-one-relationship.md).
