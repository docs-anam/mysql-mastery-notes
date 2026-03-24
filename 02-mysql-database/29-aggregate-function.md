# Aggregate Functions

## Core Functions

- `COUNT(*)`: counts rows
- `SUM(column)`: sums numeric values
- `AVG(column)`: calculates the average
- `MIN(column)`: returns the minimum value
- `MAX(column)`: returns the maximum value

## Example

```sql
SELECT
  COUNT(*) AS total_orders,
  SUM(total_amount) AS gross_amount,
  AVG(total_amount) AS average_order
FROM orders;
```

## Next Step

Continue to [30-grouping.md](30-grouping.md).

