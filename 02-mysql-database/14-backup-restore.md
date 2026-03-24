# Backup and Restore

## Why Backups Matter

A database backup is your **recovery plan**. Without it, a dropped table, a bad migration, a hardware failure, or a ransomware attack can mean permanent data loss.

```
Risk                         →  Recovery Without Backup  →  Recovery With Backup
─────────────────────────────────────────────────────────────────────────────────
Accidental DELETE all rows   →  Data is gone forever     →  Restore from last backup
Bad migration rolled to prod →  Manual reconstruction    →  Restore + replay binlogs
Server hardware failure      →  Data lost                →  Restore to new server
```

---

## mysqldump — Logical Backup

`mysqldump` is the standard MySQL backup tool. It exports the database as a SQL file containing `CREATE TABLE` + `INSERT` statements — a human-readable, portable backup.

### Backup a Single Database

```bash
mysqldump -u root -p bookstore > bookstore_backup.sql
```

### Backup with Best-Practice Flags (InnoDB)

```bash
mysqldump -u root -p \
  --single-transaction \   # Consistent snapshot without locking tables
  --routines \             # Include stored procedures and functions
  --triggers \             # Include triggers
  --events \               # Include scheduled events
  bookstore > bookstore_backup.sql
```

| Flag | Purpose |
|------|---------|
| `--single-transaction` | Uses a transaction snapshot for InnoDB — no table locks needed |
| `--routines` | Includes stored procedures and functions |
| `--triggers` | Includes table triggers |
| `--events` | Includes scheduled events |
| `--no-data` | Schema only — no row data (useful for dev environments) |
| `--no-create-info` | Data only — no CREATE TABLE statements |

### Backup All Databases

```bash
mysqldump -u root -p --all-databases \
  --single-transaction \
  --routines --triggers --events \
  > all_databases_backup.sql
```

### Backup a Specific Table

```bash
mysqldump -u root -p bookstore customers orders > customers_orders.sql
```

### Compressed Backup

```bash
# Compress on-the-fly using gzip
mysqldump -u root -p --single-transaction bookstore \
  | gzip > bookstore_$(date +%Y%m%d_%H%M%S).sql.gz
```

---

## Restore from Backup

### Restore a Full Database

```bash
# The target database must already exist
mysql -u root -p bookstore < bookstore_backup.sql
```

### Create Database and Restore in One Step

```bash
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS bookstore CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p bookstore < bookstore_backup.sql
```

### Restore a Compressed Backup

```bash
gunzip < bookstore_20260324_120000.sql.gz | mysql -u root -p bookstore
```

### Restore Verification

After restoring, always verify:

```sql
USE bookstore;
SHOW TABLES;

-- Spot-check row counts
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM customers;

-- Check the latest rows
SELECT * FROM orders ORDER BY created_at DESC LIMIT 5;
```

---

## Binary Log — Point-in-Time Recovery

`mysqldump` gives you a snapshot at a point in time. **Binary logs** (binlogs) record every change after that point, enabling you to recover to a specific moment.

### Setup

```sql
-- Check if binary logging is enabled
SHOW VARIABLES LIKE 'log_bin';
-- Should be ON for production systems

-- Find the current binary log file
SHOW BINARY LOGS;
```

### Point-in-Time Recovery Workflow

```bash
# 1. Restore the most recent full backup
mysql -u root -p bookstore < bookstore_backup_20260324.sql

# 2. Replay binary log events only up to the time before the incident
mysqlbinlog \
  --start-datetime="2026-03-24 08:00:00" \
  --stop-datetime="2026-03-24 10:30:00" \
  /var/lib/mysql/binlog.000123 | mysql -u root -p bookstore
```

---

## Automating Backups

### Daily Backup Script

```bash
#!/bin/bash
# /usr/local/bin/mysql_backup.sh

DB_NAME="bookstore"
BACKUP_DIR="/var/backups/mysql"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${TIMESTAMP}.sql.gz"

mkdir -p "$BACKUP_DIR"

mysqldump -u backup_user -p"${MYSQL_BACKUP_PASSWORD}" \
  --single-transaction --routines --triggers --events \
  "$DB_NAME" | gzip > "$BACKUP_FILE"

# Keep only the last 30 days of backups
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +30 -delete

echo "Backup complete: $BACKUP_FILE"
```

> Store the password in an environment variable or a `.my.cnf` file — never hard-code it in a script.

### Schedule with cron

```bash
# Edit the crontab
crontab -e

# Run backup at 2:00 AM every day
0 2 * * * /usr/local/bin/mysql_backup.sh >> /var/log/mysql_backup.log 2>&1
```

---

## Backup Strategy

| Frequency | Method | Recovery Point Objective |
|-----------|--------|--------------------------|
| Daily | Full `mysqldump` | Up to 24 hours of data loss possible |
| Hourly | Binary log snapshot | Up to 1 hour of data loss possible |
| Before each migration | Manual `mysqldump` | Instant rollback at any migration step |
| Weekly | Off-site or cloud copy | Disaster recovery — hardware loss |

---

## mysqldump vs Other Tools

| Tool | Type | Use Case |
|------|------|---------|
| `mysqldump` | Logical — SQL text | Standard, portable, human-readable |
| `mysqlpump` | Logical — parallel | Faster dumps for large databases |
| Percona XtraBackup | Physical — file copy | Hot backup without downtime for very large DBs |
| MySQL Enterprise Backup | Physical | Commercial tool from Oracle |

---

## Best Practices

- **Test your restores** — a backup you have never restored from is not a real backup. Verify at least monthly.
- Use `--single-transaction` for all InnoDB backups — avoids locking tables and ensures consistency.
- Store backups **off-site** or in cloud storage (S3, GCS) — a backup on the same server as the database is lost if the server fails.
- Encrypt backup files when they contain sensitive data: `... | openssl enc -aes-256-cbc -out backup.sql.gz.enc`
- Use a **dedicated backup user** with minimal privileges (`SELECT, LOCK TABLES, SHOW VIEW, EVENT`) — not root.
- Define your **RTO** (Recovery Time Objective) and **RPO** (Recovery Point Objective) before choosing a backup strategy.

## Common Mistakes

- Not testing restores — the most common and most costly mistake.
- Backing up on the same server as the database — a hardware failure destroys both.
- Using `mysqldump` without `--single-transaction` on InnoDB — produces an inconsistent dump if data changes during the dump.
- Storing the backup password in a plain-text script file tracked in version control.
- Having no automated backup schedule — relying on manual backups means they get forgotten.

## Next Step

Continue to [../03-mysql-database-acid/](../03-mysql-database-acid/) for deeper ACID and isolation-level analysis.
