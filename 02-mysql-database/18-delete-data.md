# Deleting Data

## Basic Delete

```sql
DELETE FROM customers
WHERE id = 3;
```

## Warning

`DELETE` without `WHERE` removes all rows.

## Safer Flow

```sql
SELECT * FROM customers WHERE id = 3;
DELETE FROM customers WHERE id = 3;
```

## Next Step

Continue to [19-alias.md](19-alias.md).

