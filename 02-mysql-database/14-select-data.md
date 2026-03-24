# Selecting Data

## Basic Query

```sql
SELECT id, full_name, email
FROM customers;
```

## With Expression

```sql
SELECT id, LOWER(email) AS normalized_email
FROM customers;
```

## Notes

- Avoid `SELECT *` in production queries.
- Select only needed columns.

## Next Step

Continue to [15-primary-key.md](15-primary-key.md).

