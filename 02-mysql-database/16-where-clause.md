# WHERE Clause

## Purpose

The WHERE clause filters rows so only matching records are returned or modified.

## Example

```sql
SELECT id, full_name
FROM customers
WHERE id > 10;
```

## Common Conditions

- Comparison: =, !=, >, <, >=, <=
- Range: BETWEEN
- Set: IN
- Pattern: LIKE
- Null checks: IS NULL, IS NOT NULL

## Next Step

Continue to [17-update-data.md](17-update-data.md).
