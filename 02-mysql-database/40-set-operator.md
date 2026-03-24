# Set Operators

## UNION and UNION ALL

- `UNION` removes duplicates
- `UNION ALL` keeps duplicates

## Example

```sql
SELECT email FROM customers_archive
UNION
SELECT email FROM customers;
```

## Rule

Every query in a set operation must return the same number of columns with compatible data types.

## Next Step

Continue to [41-transaction.md](41-transaction.md).

