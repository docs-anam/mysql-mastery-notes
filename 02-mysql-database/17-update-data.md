# Updating Data

## Basic Update

```sql
UPDATE customers
SET full_name = 'Budi Santoso'
WHERE id = 2;
```

## Safety Checklist

- Always test condition with `SELECT` first.
- Use a precise `WHERE` clause.
- Use transactions for critical updates.

## Next Step

Continue to [18-delete-data.md](18-delete-data.md).

