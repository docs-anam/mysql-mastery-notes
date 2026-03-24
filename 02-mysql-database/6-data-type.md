# MySQL Data Types Overview

## Main Families

- Numeric: `INT`, `BIGINT`, `DECIMAL`
- String: `VARCHAR`, `TEXT`
- Date and time: `DATE`, `DATETIME`, `TIMESTAMP`
- Logical: `BOOLEAN` (alias of `TINYINT(1)`)
- Specialized: `JSON`, `ENUM`, `BLOB`

## Example

```sql
CREATE TABLE products (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(120) NOT NULL,
  price DECIMAL(12,2) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Next Step

Continue to [7-data-type-number.md](7-data-type-number.md).

