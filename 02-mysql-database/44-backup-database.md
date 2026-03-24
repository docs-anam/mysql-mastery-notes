# Backup Database

## Logical Backup

```bash
mysqldump -u root -p bookstore > bookstore_backup.sql
```

## Best Practices

- Schedule backups periodically.
- Store backups in a separate location.
- Encrypt sensitive backups.
- Test backup integrity and restore procedures.

## Next Step

Continue to [45-restore-database.md](45-restore-database.md).

