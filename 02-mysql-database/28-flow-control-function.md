# Flow Control Functions

## CASE Expression

```sql
SELECT
  id,
  total_amount,
  CASE
    WHEN total_amount >= 1000000 THEN 'high'
    WHEN total_amount >= 300000 THEN 'medium'
    ELSE 'low'
  END AS order_segment
FROM orders;
```

## IF Function

```sql
SELECT IF(is_active = 1, 'active', 'inactive') AS status
FROM users;
```

## Next Step

Continue to [29-aggregate-function.md](29-aggregate-function.md).
