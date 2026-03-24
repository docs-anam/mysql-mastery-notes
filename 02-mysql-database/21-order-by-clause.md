# ORDER BY Clause

## Basic Sorting

```sql
SELECT id, full_name, created_at
FROM customers
ORDER BY created_at DESC;
```

## Multi Column Sort

```sql
ORDER BY is_active DESC, full_name ASC
```

## Next Step

Continue to [22-limit-clause.md](22-limit-clause.md).

