# User Management

## Create User

```sql
CREATE USER 'app_user'@'%' IDENTIFIED BY 'strong_password';
```

Replace `'strong_password'` with a secure password from your secret manager or environment.

## Grant Privileges

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON bookstore.*
TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

## Revoke Privileges

```sql
REVOKE DELETE ON bookstore.* FROM 'app_user'@'%';
```

## Next Step

Continue to [44-backup-database.md](44-backup-database.md).

