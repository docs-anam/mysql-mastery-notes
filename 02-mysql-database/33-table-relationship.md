# Table Relationships

## Relationship Types

- One-to-One
- One-to-Many
- Many-to-Many

## Why Relationships Matter

Relationships preserve data integrity and reduce redundancy through foreign keys and proper table boundaries.

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

Continue to [34-join.md](34-join.md).
