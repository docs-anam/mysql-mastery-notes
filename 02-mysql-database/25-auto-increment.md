# AUTO_INCREMENT

## What It Does

`AUTO_INCREMENT` generates sequential numeric values, commonly for primary keys.

## Example

```sql
CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  order_number VARCHAR(40) NOT NULL UNIQUE
);
```

## Notes

- Each insert without an explicit `id` gets the next value.
- Gaps may appear after rollback or delete, and this is normal.
- Avoid using auto increment as a strict business sequence if no gaps are allowed.

## Next Step

Continue to [26-string-function.md](26-string-function.md).
