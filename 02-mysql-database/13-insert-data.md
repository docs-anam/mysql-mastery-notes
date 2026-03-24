# Inserting Data

## Single Row Insert

```sql
INSERT INTO customers (full_name, email)
VALUES ('Ana Putri', 'ana@example.com');
```

## Multi Row Insert

```sql
INSERT INTO customers (full_name, email)
VALUES
  ('Budi Santoso', 'budi@example.com'),
  ('Citra Dewi', 'citra@example.com');
```

## Insert Best Practices

- Always specify column names explicitly.
- Validate data before insert.
- Use transactions for grouped inserts.

## Next Step

Continue to [14-select-data.md](14-select-data.md).
