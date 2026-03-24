# Numeric Data Types

## Common Types

- `TINYINT`: small integer
- `INT`: standard integer
- `BIGINT`: large integer
- `DECIMAL(p,s)`: fixed-precision decimal
- `FLOAT`, `DOUBLE`: approximate precision

## Recommendation

Use `DECIMAL` for financial values and `INT` or `BIGINT` for identifiers.

## Example

```sql
CREATE TABLE invoices (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  total_amount DECIMAL(14,2) NOT NULL
);
```

## Next Step

Continue to [8-data-type-string.md](8-data-type-string.md).

