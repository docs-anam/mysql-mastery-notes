# One-to-Many Relationship

## Definition

One row in a parent table is related to many rows in a child table.

## Example

```sql
CREATE TABLE customers (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  full_name VARCHAR(120) NOT NULL
);

CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  customer_id BIGINT NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

## Next Step

Continue to [37-many-to-many-relationship.md](37-many-to-many-relationship.md).

