# Date and Time Data Types

## Types

- `DATE` date only
- `TIME` time only
- `DATETIME` date and time
- `TIMESTAMP` date and time with timezone-aware conversion behavior

## Example

```sql
CREATE TABLE events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  event_name VARCHAR(120) NOT NULL,
  event_start DATETIME NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Next Step

Continue to [10-data-type-boolean.md](10-data-type-boolean.md).

