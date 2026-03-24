# Restore Database

## Restore from SQL Dump

```bash
mysql -u root -p bookstore < bookstore_backup.sql
```

## Verification Checklist

- Confirm tables are restored.
- Validate row counts for critical tables.
- Run smoke-test queries.
- Check foreign key consistency.

## Practical Advice

Restoration is only reliable if it is tested periodically. A backup without restore testing is an unverified assumption.

## Next Step

You can continue to [03-mysql-database-acid](../03-mysql-database-acid/) for deeper ACID and transaction isolation study.
