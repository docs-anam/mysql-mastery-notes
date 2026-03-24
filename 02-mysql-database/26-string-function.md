# String Functions

## Common Functions

- `CONCAT(a, b)` concatenate strings
- `LOWER(s)` lowercase
- `UPPER(s)` uppercase
- `TRIM(s)` remove surrounding spaces
- `LENGTH(s)` length in bytes
- `CHAR_LENGTH(s)` length in characters

## Example

```sql
SELECT
  full_name,
  CONCAT(full_name, ' <', email, '>') AS contact
FROM customers;
```

## Next Step

Continue to [27-date-and-time-function.md](27-date-and-time-function.md).

