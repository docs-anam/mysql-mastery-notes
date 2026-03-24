# Creating and Managing Tables

## Create Table

```sql
CREATE TABLE customers (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  full_name VARCHAR(120) NOT NULL,
  email VARCHAR(120) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Inspect Table

```sql
DESCRIBE customers;
SHOW CREATE TABLE customers;
```

## Next Step

Continue to [13-insert-data.md](13-insert-data.md).

