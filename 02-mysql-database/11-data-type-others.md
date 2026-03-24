# Other Useful MySQL Data Types

## JSON

Use `JSON` for semi-structured configuration or metadata.

```sql
CREATE TABLE user_preferences (
  user_id BIGINT PRIMARY KEY,
  preferences JSON NOT NULL
);
```

## ENUM

Use `ENUM` for a small, fixed set of values.

```sql
status ENUM('pending', 'paid', 'cancelled') NOT NULL
```

## Next Step

Continue to [12-table.md](12-table.md).

