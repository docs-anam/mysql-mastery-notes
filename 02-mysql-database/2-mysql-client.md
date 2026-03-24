# Entering the MySQL Database System

## What is the MySQL Client?

The **MySQL client** (`mysql` CLI) is a command-line tool that lets you connect to a MySQL server and execute SQL statements interactively. When you type a query and press Enter, the client sends it to the server, which processes it and sends back results.

```
┌─────────────────┐       TCP/IP or Unix Socket       ┌────────────────────┐
│   MySQL Client  │ ─────────────────────────────────▶│   MySQL Server     │
│   (mysql CLI)   │ ◀─────────────────────────────────│   (mysqld daemon)  │
└─────────────────┘           Results                 └────────────────────┘
```

## Connecting to MySQL

### Basic Connection

```bash
mysql -u root -p
```

| Flag | Meaning | Default |
|------|---------|---------|
| `-u` | Username | — |
| `-p` | Prompt for password | — |
| `-h` | Hostname | `localhost` |
| `-P` | Port number | `3306` |
| `-D` | Default database to select | — |

### Connect to a Specific Database

```bash
mysql -u root -p bookstore
```

### Connect to a Remote Host

```bash
mysql -u app_user -p -h 192.168.1.10 -P 3306
```

### Connection String (DSN Format)

Used by applications and frameworks:

```
mysql://username:password@hostname:port/database
mysql://app_user:secret@localhost:3306/bookstore
```

## Basic Session Commands

Once connected, these are the most important commands:

```sql
-- List all databases on the server
SHOW DATABASES;

-- Select a database to work in
USE bookstore;

-- Check which database is currently active
SELECT DATABASE();

-- Show the current MySQL version
SELECT VERSION();

-- Check who you are logged in as
SELECT USER();

-- List all tables in the active database
SHOW TABLES;

-- Show columns and types for a table
DESCRIBE customers;

-- Exit the session
EXIT;
-- or
QUIT;
```

## Session Flow Example

```
Step 1 → Connect          :  mysql -u root -p
Step 2 → Authenticate     :  Enter password
Step 3 → List databases   :  SHOW DATABASES;
Step 4 → Select database  :  USE bookstore;
Step 5 → Verify context   :  SELECT DATABASE();
Step 6 → Work with data   :  SELECT, INSERT, UPDATE, DELETE...
Step 7 → Exit             :  EXIT;
```

## MySQL Server Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        MySQL Server                         │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │          Connection Handler / Thread Pool             │  │
│  │  (Manages one thread per client connection)           │  │
│  └──────────────────────────┬────────────────────────────┘  │
│                             │                               │
│  ┌──────────────────────────▼────────────────────────────┐  │
│  │         SQL Parser & Query Optimizer                  │  │
│  │  (Validates syntax, picks best execution plan)        │  │
│  └──────────────────────────┬────────────────────────────┘  │
│                             │                               │
│  ┌──────────────────────────▼────────────────────────────┐  │
│  │            InnoDB Storage Engine                      │  │
│  │  (Reads/writes data, manages transactions & locks)    │  │
│  └──────────────────────────┬────────────────────────────┘  │
│                             │                               │
└─────────────────────────────┼───────────────────────────────┘
                              │
                 ┌────────────▼────────────┐
                 │   Physical Storage      │
                 │   (.ibd files on disk)  │
                 └─────────────────────────┘
```

## Default System Databases

When MySQL is first installed, these databases exist:

| Database | Purpose |
|----------|---------|
| `mysql` | User accounts, privileges, and server configuration |
| `information_schema` | Metadata views — describes all databases, tables, columns |
| `performance_schema` | Runtime server performance metrics |
| `sys` | Human-readable views on top of `performance_schema` |

> **Important**: Never manually modify the `mysql` system tables. Always use `CREATE USER`, `GRANT`, and `REVOKE` statements instead.

## Exploring an Unfamiliar Database

When working with a database you did not design, use these commands to explore:

```sql
-- See all tables
SHOW TABLES;

-- See columns for a specific table
DESCRIBE orders;

-- See the full CREATE TABLE definition
SHOW CREATE TABLE orders;

-- Count rows in a table
SELECT COUNT(*) FROM orders;

-- Show all indexes on a table
SHOW INDEX FROM orders;

-- View current server status
STATUS;
```

## Graphical Interfaces

Besides the CLI, these GUI tools offer a visual way to work with MySQL:

| Tool | Type | Best For |
|------|------|----------|
| MySQL Workbench | Desktop | Full-featured official tool, schema design |
| TablePlus | Desktop | Clean UI, fast query editor |
| DBeaver | Desktop | Open-source, supports many database types |
| phpMyAdmin | Web | Browser-based, common in shared hosting |
| DataGrip | IDE plugin | JetBrains product, intelligent SQL completion |

## Best Practices

- **Never** use the `root` account for routine application access — create a dedicated user with minimal privileges.
- Always verify the active database with `SELECT DATABASE()` before running any data modification.
- Use `DESCRIBE <table>` and `SHOW CREATE TABLE <table>` to understand a schema before writing queries.
- In applications, use a **connection pool** instead of opening a new connection per request.

## Common Mistakes

- Forgetting to run `USE <database>` before querying — causes `No database selected` errors.
- Connecting as `root` from application code — this is a serious security risk.
- Assuming `localhost` connects via TCP; MySQL defaults to a Unix socket on the same host. Use `127.0.0.1` to force TCP if needed.

## Next Step

Continue to [3-intro-sql.md](3-intro-sql.md) to understand the categories and anatomy of SQL.

