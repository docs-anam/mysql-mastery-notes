# User and Permission Management

## Why This Matters

MySQL's access control system is your last line of defence against accidental or malicious data changes. A misconfigured user account can expose your entire database.

```
Application  →  app_user  (SELECT, INSERT, UPDATE, DELETE only)
Reports      →  report_user  (SELECT only)
Migrations   →  admin_user  (ALL PRIVILEGES — temporary)
Root         →  never used by applications
```

---

## How MySQL Access Control Works

Every connection to MySQL is authenticated by a **user account** identified by a username and host pair. The host restricts **where** the user may connect from.

```
'username'@'host'

'app_user'@'localhost'     → only from the same machine
'app_user'@'192.168.1.%'  → from any IP in 192.168.1.0/24
'app_user'@'%'             → from anywhere (less secure)
```

Authentication → then → Authorization (privilege check) → then → Query execution

---

## Creating Users

```sql
-- User that can only connect locally
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'StrongP@ssword1!';

-- User that can connect from any host (e.g., an app server)
CREATE USER 'app_user'@'%' IDENTIFIED BY 'StrongP@ssword1!';

-- Safe creation: no error if user already exists
CREATE USER IF NOT EXISTS 'report_user'@'localhost' IDENTIFIED BY 'ReportP@ss!';
```

### Password Policy

MySQL 8 enforces a default password policy:
- Minimum 8 characters
- Mix of uppercase, lowercase, digits, and special characters

```sql
-- Check the current password policy
SHOW VARIABLES LIKE 'validate_password%';
```

---

## Granting Privileges

```sql
-- Syntax
GRANT privilege_list ON scope TO 'user'@'host';

-- Standard application user (read + write on one database)
GRANT SELECT, INSERT, UPDATE, DELETE ON bookstore.* TO 'app_user'@'localhost';

-- Read-only user (for reporting tools, dashboards)
GRANT SELECT ON bookstore.* TO 'report_user'@'localhost';

-- Full access on one database (for migration tools — temporary)
GRANT ALL PRIVILEGES ON bookstore.* TO 'admin_user'@'localhost';

-- Specific table only
GRANT SELECT ON bookstore.products TO 'catalog_reader'@'localhost';

-- Apply changes immediately
FLUSH PRIVILEGES;
```

### Privilege Scope Options

| Scope | Syntax | Effect |
|-------|--------|--------|
| All databases | `*.*` | Global — very broad — avoid for app users |
| One database | `dbname.*` | All tables in the database |
| One table | `dbname.tablename` | Only that table |
| One column | `dbname.table(col)` | Only that column |

---

## Viewing Grants

```sql
-- See all grants for a user
SHOW GRANTS FOR 'app_user'@'localhost';

-- See grants for the currently connected user
SHOW GRANTS;

-- List all user accounts on the server
SELECT user, host FROM mysql.user;
```

---

## Revoking Privileges

```sql
-- Remove a specific privilege
REVOKE DELETE ON bookstore.* FROM 'app_user'@'localhost';

-- Remove all privileges on a database
REVOKE ALL PRIVILEGES ON bookstore.* FROM 'app_user'@'localhost';

-- Always flush after changes
FLUSH PRIVILEGES;
```

---

## Changing a Password

```sql
-- Change another user's password (requires privileges)
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'NewStrongP@ss!';

-- Change your own password
ALTER USER CURRENT_USER() IDENTIFIED BY 'NewStrongP@ss!';
```

---

## Dropping a User

```sql
DROP USER 'app_user'@'localhost';

-- Safe version
DROP USER IF EXISTS 'app_user'@'localhost';
```

> Dropping a user automatically revokes all their privileges.

---

## Principle of Least Privilege

**Give each user account only the minimum permissions required for its purpose.**

| User Type | Recommended Privileges | Notes |
|-----------|----------------------|-------|
| Application (read/write) | `SELECT, INSERT, UPDATE, DELETE` on specific DB | Most common |
| Read-only / reporting | `SELECT` only | BI tools, dashboards |
| Migration runner | `ALL PRIVILEGES` on specific DB | Temporary — revoke after migration |
| Backup user | `SELECT, LOCK TABLES, SHOW VIEW, EVENT` | Minimal for `mysqldump` |
| Replication user | `REPLICATION SLAVE, REPLICATION CLIENT` | For replica setup only |

> **Never** grant `GRANT OPTION`, `DROP`, `CREATE`, or `FILE` to routine application users.

---

## Practical Setup Example

```sql
-- 1. Create the application user (local only)
CREATE USER 'bookstore_app'@'localhost' IDENTIFIED BY 'AppP@ssword42!';

-- 2. Create a separate read-only reporting user
CREATE USER 'bookstore_report'@'localhost' IDENTIFIED BY 'R3portP@ss!';

-- 3. Grant appropriate privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON bookstore.* TO 'bookstore_app'@'localhost';
GRANT SELECT ON bookstore.* TO 'bookstore_report'@'localhost';
FLUSH PRIVILEGES;

-- 4. Verify
SHOW GRANTS FOR 'bookstore_app'@'localhost';
SHOW GRANTS FOR 'bookstore_report'@'localhost';
```

---

## Best Practices

- **Never** use `root` for application connections. Create a dedicated user with minimal privileges.
- Restrict the host to `localhost` or a specific IP range whenever possible — avoid `'%'` unless required.
- Use strong, unique passwords per environment (dev, staging, production should have different credentials).
- Revoke migration-user privileges immediately after a migration is complete.
- Audit user accounts periodically: `SELECT user, host FROM mysql.user;`

## Common Mistakes

- Using the same `app_user` account for both the application and database migrations — migrations need `ALTER TABLE` and `CREATE TABLE`, regular app code should not.
- Granting `ALL PRIVILEGES ON *.*` — this gives global access to every database on the server.
- Reusing passwords across environments — a leaked development credential should not work in production.
- Forgetting `FLUSH PRIVILEGES` after manual `INSERT`/`DELETE` on `mysql.user` (always use `GRANT`/`REVOKE` instead).

## Next Step

Continue to [14-backup-restore.md](14-backup-restore.md) to learn how to back up and restore your MySQL databases safely.
