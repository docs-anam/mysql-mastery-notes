# Introduction to SQL

## SQL Categories

- DDL: `CREATE`, `ALTER`, `DROP`
- DML: `INSERT`, `UPDATE`, `DELETE`
- DQL: `SELECT`
- TCL: `COMMIT`, `ROLLBACK`
- DCL: `GRANT`, `REVOKE`

## Example

```sql
SELECT id, full_name
FROM customers
WHERE is_active = 1
ORDER BY created_at DESC
LIMIT 10;
```

## Next Step

Continue to [4-install-mysql.md](4-install-mysql.md).

