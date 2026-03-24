# Date and Time Functions

## Useful Functions

- `NOW()` current date and time
- `CURDATE()` current date
- `DATE_ADD(dt, INTERVAL n DAY)` add interval
- `DATEDIFF(d1, d2)` difference in days
- `DATE_FORMAT(dt, format)` custom display format

## Example

```sql
SELECT
  id,
  created_at,
  DATE_FORMAT(created_at, '%Y-%m-%d') AS created_date
FROM orders;
```

## Next Step

Continue to [28-flow-control-function.md](28-flow-control-function.md).
