# Numeric Functions

## Common Functions

- `ABS(x)`: returns the absolute value
- `ROUND(x, d)`: rounds to `d` decimal places
- `CEIL(x)`: rounds up to the nearest integer
- `FLOOR(x)`: rounds down to the nearest integer
- `MOD(a, b)`: returns the remainder

## Example

```sql
SELECT
  price,
  ROUND(price, 2) AS rounded_price,
  CEIL(price) AS ceil_price
FROM products;
```

## Next Step

Continue to [25-auto-increment.md](25-auto-increment.md).
