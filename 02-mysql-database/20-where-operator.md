# Operators for WHERE Clause

## Logical Operators

- AND: all conditions must be true
- OR: at least one condition must be true
- NOT: negate a condition

## Example

```sql
SELECT id, full_name
FROM customers
WHERE is_active = 1
  AND created_at >= '2026-01-01';
```

## Additional Examples

```sql
WHERE full_name LIKE 'A%'
WHERE id IN (1, 2, 3)
```

## Next Step

Continue to [21-order-by-clause.md](21-order-by-clause.md).
