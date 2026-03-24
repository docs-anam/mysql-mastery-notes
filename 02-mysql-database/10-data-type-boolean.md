# Boolean Data Type in MySQL

## Important Note

`BOOLEAN` is an alias for `TINYINT(1)` in MySQL.

## Convention

- `0` means false
- `1` means true

## Example

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(120) NOT NULL UNIQUE,
  is_active BOOLEAN NOT NULL DEFAULT 1
);
```

## Next Step

Continue to [11-data-type-others.md](11-data-type-others.md).

