# Entering the MySQL Database System

## Basic Session Flow

1. Connect to the MySQL server
2. Authenticate with a user account
3. List available databases
4. Select the database context with `USE`

## Example

```sql
SHOW DATABASES;
USE bookstore;
SELECT DATABASE();
```

## Best Practices

- Avoid using the root account for routine application access.
- Verify active database before any data changes.

## Next Step

Continue to [3-intro-sql.md](3-intro-sql.md).

