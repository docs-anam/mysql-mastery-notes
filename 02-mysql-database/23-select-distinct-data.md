# SELECT DISTINCT

## Use Case

`DISTINCT` removes duplicate rows from the selected columns.

## Example

```sql
SELECT DISTINCT city
FROM customers;
```

## Multi Column Distinct

```sql
SELECT DISTINCT city, country
FROM customers;
```

## Caution

`DISTINCT` can be expensive on very large datasets. Use with proper indexing and only when needed.

## Next Step

Continue to [24-numeric-function.md](24-numeric-function.md).
