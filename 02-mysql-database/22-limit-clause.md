# LIMIT Clause

## Purpose

`LIMIT` controls how many rows are returned.

## Example

```sql
SELECT id, full_name
FROM customers
ORDER BY id DESC
LIMIT 10;
```

## Pagination Pattern

```sql
LIMIT 10 OFFSET 20
```

## Next Step

Continue to [23-select-distinct-data.md](23-select-distinct-data.md).

