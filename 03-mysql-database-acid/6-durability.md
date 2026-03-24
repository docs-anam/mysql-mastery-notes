# Durability

## What is Durability?

**Durability** guarantees that once a transaction is committed, it is **permanently recorded** — even if the server crashes immediately afterwards. A committed transaction will never be lost.

> "A COMMIT is a promise. Whatever happens next — power failure, kernel panic, hardware crash — the data is safe."

---

## The Problem Durability Solves

Without durability guarantees, a crash after a successful `COMMIT` could lose the transaction:

```
User's browser:     Transfer ₹5,000 from Alice to Bob
MySQL returns:      Query OK — COMMIT successful
Power fails:        Server crashes immediately
Server restarts:    ...is the transfer still there?

WITHOUT durability: Alice's ₹5000 is gone, Bob got nothing — data is lost
WITH durability:    Transfer is fully present on restart
```

---

## How InnoDB Ensures Durability

InnoDB uses three mechanisms working together:

```
Application commits a transaction
           │
           ▼
┌─────────────────────────────┐
│  1. Write to Redo Log (WAL) │  ← written to disk BEFORE commit returns
│     (innodb_log_files)      │
└─────────────────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  2. fsync() called          │  ← OS confirms bytes are on disk, not in write cache
└─────────────────────────────┘
           │
           ▼
   COMMIT returns to client   ← only now does MySQL respond "OK"
           │
           ▼ (async, later)
┌─────────────────────────────┐
│  3. Buffer Pool dirty pages │  ← actual data pages written to tablespace (.ibd)
│     flushed to .ibd files   │
└─────────────────────────────┘
```

---

## Key Components

### 1. Redo Log (Write-Ahead Log)

The **redo log** is a fixed-size circular file on disk that records every physical change InnoDB makes. Before any data page is modified in memory, the change is appended to the redo log.

```
Redo log file: iblogfile0, iblogfile1  (default 48MB each)

Each log record contains:
- LSN (Log Sequence Number) — a monotonically increasing position
- Table space ID and page number
- The actual change (e.g., "set byte offset 120 of page 44 to value X")

On COMMIT: InnoDB calls fsync() on the redo log → data is safe
```

### 2. InnoDB Buffer Pool

The buffer pool is an in-memory cache of data pages. Reads and writes go through the buffer pool first.

```
Disk (tablespace .ibd files)
        │  read page
        ▼
┌──────────────────────────┐
│      Buffer Pool         │  ← in-memory, fast
│  [Page 1] [Page 5] ...   │  dirty pages are written back asynchronously
└──────────────────────────┘
        │  flush dirty page (async)
        ▼
Disk (tablespace .ibd files)
```

> Because writes to data pages are async, a crash could leave data pages incomplete. The **redo log** is what makes recovery possible.

### 3. Crash Recovery

When MySQL restarts after a crash, InnoDB automatically replays the redo log:

```
Server crashes at time T
         │
         ▼
MySQL restarts → InnoDB reads redo log
         │
         ├── Committed transactions with redo record: REPLAYED → data applied to pages
         └── Uncommitted transactions (no COMMIT in log): ROLLED BACK → discarded
         │
         ▼
Database is consistent as of the last committed transaction
```

---

## `innodb_flush_log_at_trx_commit`

This variable controls **when** InnoDB writes and flushes the redo log. It is the single most important durability knob.

| Value | Write Behavior | fsync Behavior | Durability | Performance |
|-------|---------------|---------------|-----------|------------|
| `0` | Flushed every 1 second by background thread | Every 1 second | ❌ Up to 1 second of data loss on crash | Highest |
| `1` | Written and flushed at every `COMMIT` | At every `COMMIT` | ✅ Full (D in ACID) — default | Moderate |
| `2` | Written at every `COMMIT`, flushed every 1 second | Every 1 second | ⚠️ Safe if only OS crashes; not if power fails | High |

```sql
-- View current setting
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';

-- Change for current session (testing only)
SET GLOBAL innodb_flush_log_at_trx_commit = 1;

-- Or set permanently in my.cnf:
-- [mysqld]
-- innodb_flush_log_at_trx_commit = 1
```

> **Rule of thumb**: Use `1` for any data you cannot afford to lose. Use `2` on a system with a UPS (uninterruptible power supply) for better write throughput.

---

## `sync_binlog` — Binary Log Durability

The **binary log** records every committed transaction for replication and point-in-time recovery. Its durability is controlled separately.

| Value | Behavior |
|-------|---------|
| `0` | MySQL never syncs the binlog; OS decides when to flush (fast, risky) |
| `1` | fsync() after every transaction commit — fully durable (recommended) |
| `N` | fsync() after every N transactions |

```sql
SHOW VARIABLES LIKE 'sync_binlog';
-- Recommended production setting: sync_binlog = 1
```

For **full ACID durability** on a single server, use:

```ini
[mysqld]
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

For **replica servers** prioritizing throughput over crash safety:

```ini
[mysqld]
innodb_flush_log_at_trx_commit = 2
sync_binlog = 0
```

---

## Demonstrating Durability

### Test: Does a commit survive a server restart?

```sql
USE bank;

-- Insert a record
START TRANSACTION;
INSERT INTO accounts (owner_name, balance, is_active)
VALUES ('DurabilityTest', 999999, TRUE);
COMMIT;

-- Verify it exists
SELECT * FROM accounts WHERE owner_name = 'DurabilityTest';
```

Now **stop and restart MySQL** (simulate a crash):

```bash
# On macOS (Homebrew)
brew services stop mysql
brew services start mysql

# On Ubuntu/Debian
sudo systemctl stop mysql
sudo systemctl start mysql

# On Docker
docker restart mysql_container
```

Reconnect and verify:

```sql
USE bank;
SELECT * FROM accounts WHERE owner_name = 'DurabilityTest';
-- The row must still be here
```

### Test: Uncommitted data does NOT survive

```sql
START TRANSACTION;
INSERT INTO accounts (owner_name, balance, is_active) VALUES ('LostData', 100, TRUE);
-- Do NOT commit — instead, kill the server
```

After restart:

```sql
SELECT * FROM accounts WHERE owner_name = 'LostData';
-- Empty — uncommitted data is gone (as expected)
```

---

## Point-in-Time Recovery with Binary Log

The binary log allows you to recover to any point in time after a backup.

### Enable binary logging (my.cnf)

```ini
[mysqld]
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
expire_logs_days = 7
```

### Recovery Workflow

```bash
# Step 1: Restore from the most recent full backup
mysql -u root -p < backup_20241001.sql

# Step 2: Find the binary log position at the time of the failure
mysqlbinlog /var/log/mysql/mysql-bin.000042 | grep -A 2 "2024-10-01 14:30"

# Step 3: Replay binary logs up to that point (stop-datetime)
mysqlbinlog \
  --start-datetime="2024-10-01 00:00:00" \
  --stop-datetime="2024-10-01 14:29:59" \
  /var/log/mysql/mysql-bin.000042 \
  /var/log/mysql/mysql-bin.000043 \
  | mysql -u root -p

# Database is now restored to the moment just before the failure
```

---

## InnoDB Doublewrite Buffer

InnoDB writes data pages to a **doublewrite buffer** on disk before writing them to their final location in the tablespace. This protects against **partial page writes** (a 16KB page written only half-way before a crash).

```
Page to write
     │
     ├──► Doublewrite buffer (sequential, fast)
     │         │
     │         ▼ (only after doublewrite is synced)
     └──► Actual tablespace page location

On recovery: if the tablespace page is corrupt, InnoDB reads the intact
copy from the doublewrite buffer
```

---

## What Durability Does NOT Protect Against

| Threat | Protected by Durability? | Solution |
|--------|--------------------------|---------|
| Server crash / power failure | ✅ Yes | Redo log + fsync |
| OS kernel crash | ✅ Yes (with `flush=1`) | Redo log |
| Disk hardware failure | ❌ No | RAID, disk redundancy |
| Accidental `DELETE` or `DROP TABLE` | ❌ No | Point-in-time recovery, backups |
| Ransomware / data corruption | ❌ No | Offsite backups |
| Human error (wrong WHERE clause) | ❌ No | Backups + careful queries |

---

## Best Practices

- Always run production MySQL with `innodb_flush_log_at_trx_commit = 1` and `sync_binlog = 1`.
- Enable the binary log on all production servers — it enables point-in-time recovery.
- Test crash recovery at least once in a staging environment before relying on it in production.
- Schedule regular full backups (daily) and keep binary logs for at least 7 days.
- Monitor disk I/O — the redo log is write-intensive; put it on a fast SSD if possible.

## Common Mistakes

- Setting `innodb_flush_log_at_trx_commit = 0` on a production server for "better performance" — risks losing up to 1 second of transactions.
- Disabling the binary log — you lose replication capability and point-in-time recovery.
- Not testing the restore procedure — a backup you have never tested is not a backup.
- Storing backups on the same disk as the database — a single disk failure destroys both.

---

## ACID Module Complete

You have now covered all four ACID properties:

| Property | File | Core Mechanism |
|---------|------|---------------|
| **Atomicity** | `3-atomicity.md` | Undo log — all-or-nothing |
| **Consistency** | `4-consistency.md` | Constraints + triggers |
| **Isolation** | `5-isolation.md` | MVCC + locking |
| **Durability** | `6-durability.md` | Redo log + fsync |

## Next Step

Continue to [04-database-design-notification](../04-database-design-notification/) for applied database design patterns.