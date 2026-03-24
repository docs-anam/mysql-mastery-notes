# String Data Types

## Common Types

- `CHAR(n)` fixed-length
- `VARCHAR(n)` variable-length
- `TEXT` long text
- `BLOB` binary data

## Example

```sql
CREATE TABLE notes (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(160) NOT NULL,
  content TEXT NOT NULL
);
```

## Next Step

Continue to [9-data-type-date-and-time.md](9-data-type-date-and-time.md).

