# Primary Key

## Definition

A primary key uniquely identifies each row in a table.

## Rules

- Must be unique
- Must not be NULL
- A table has one primary key definition

## Example

```sql
CREATE TABLE categories (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(80) NOT NULL UNIQUE
);
```

## Composite Primary Key Example

```sql
PRIMARY KEY (order_id, product_id)
```

## Next Step

Continue to [16-where-clause.md](16-where-clause.md).
