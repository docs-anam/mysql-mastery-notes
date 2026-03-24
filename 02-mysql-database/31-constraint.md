# Constraints

## Common Constraint Types

- `PRIMARY KEY` unique row identity
- `FOREIGN KEY` referential integrity
- `UNIQUE` no duplicate values
- `NOT NULL` required value
- `CHECK` validation rule (depends on version)
- `DEFAULT` fallback value

## Example

```sql
CREATE TABLE order_items (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  order_id BIGINT NOT NULL,
  quantity INT NOT NULL CHECK (quantity > 0),
  FOREIGN KEY (order_id) REFERENCES orders(id)
);
```

## Next Step

Continue to [32-index.md](32-index.md).
