# Working with Databases

## What is a Database (Schema) in MySQL?

In MySQL, a **database** (also called a **schema**) is a named container that groups related tables, views, procedures, and other objects together. One MySQL server can host many databases simultaneously.

```
MySQL Server
├── mysql               (system: users, privileges)
├── information_schema  (system: metadata)
├── bookstore           ← your application database
│   ├── customers
│   ├── products
│   └── orders
└── analytics           ← another application database
    ├── sales_summary
    └── user_events
```

## Creating a Database

### Basic Syntax

```sql
CREATE DATABASE bookstore;
```

### With Character Set and Collation (Recommended)

```sql
CREATE DATABASE bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

- **`utf8mb4`** — True 4-byte Unicode; supports all characters including emoji. Use this, not `utf8`.
- **`utf8mb4_unicode_ci`** — Case-insensitive comparison using Unicode rules.

### Safe Creation (Idempotent)

```sql
CREATE DATABASE IF NOT EXISTS bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

> The `IF NOT EXISTS` guard prevents an error if the database already exists. Use this in migration scripts.

## Listing Databases

```sql
SHOW DATABASES;
```

Example output:

```
+--------------------+
| Database           |
+--------------------+
| bookstore          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

## Selecting a Database

```sql
USE bookstore;
```

After running `USE`, all subsequent table operations run against `bookstore` unless a full `database.table` reference is used.

## Checking the Active Database

```sql
SELECT DATABASE();
```

Output:

```
+------------+
| DATABASE() |
+------------+
| bookstore  |
+------------+
```

> Always verify the active database before running any DDL or DML in scripts.

## Viewing Database Details

```sql
-- Show the full CREATE DATABASE statement MySQL uses for this database
SHOW CREATE DATABASE bookstore;
```

Output includes the character set and collation configured at creation time.

## Modifying a Database

```sql
-- Change character set or collation on an existing database
ALTER DATABASE bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
```

> **Note**: Changing the database collation only affects new tables. Existing tables retain their original collation until altered individually.

## Dropping a Database

```sql
-- Remove a database and ALL its contents permanently
DROP DATABASE bookstore;

-- Safe version: no error if the database doesn't exist
DROP DATABASE IF EXISTS bookstore;
```

> **Warning**: `DROP DATABASE` is irreversible. All tables, data, and objects inside are deleted. Always back up first.

## Character Set and Collation

### Character Set

Defines which characters can be stored.

| Character Set | Description |
|--------------|-------------|
| `latin1` | Western European characters only — 1 byte per char (legacy) |
| `utf8` | MySQL's 3-byte "UTF-8" — **cannot store emoji** (misleadingly named) |
| `utf8mb4` | True 4-byte UTF-8 — supports all Unicode including emoji ✅ |

### Collation

Defines how characters are compared and sorted.

| Collation | Behavior |
|-----------|---------|
| `utf8mb4_unicode_ci` | Case-insensitive, standard Unicode rules |
| `utf8mb4_bin` | Case-sensitive, binary byte-by-byte comparison |
| `utf8mb4_0900_ai_ci` | MySQL 8 default — accent-insensitive, case-insensitive, fast |

### Check Available Collations

```sql
SHOW COLLATION WHERE Charset = 'utf8mb4' LIMIT 10;
```

## Practical Example: Bootstrap a Bookstore Database

```sql
-- 1. Create the database with proper encoding
CREATE DATABASE IF NOT EXISTS bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- 2. Select it as the active database
USE bookstore;

-- 3. Confirm the selection
SELECT DATABASE();

-- 4. View its full definition
SHOW CREATE DATABASE bookstore;
```

## Best Practices

- **Always use `utf8mb4`** — using `utf8` silently drops data for 4-byte characters like emoji.
- Name databases with **lowercase and underscores**: `my_app_db`, not `MyAppDB`.
- Use `IF NOT EXISTS` and `IF EXISTS` in scripts so they are **idempotent** (safe to run multiple times).
- Keep **one application per database** — don't mix unrelated systems in a single schema.
- Set character set and collation explicitly at creation time — don't rely on server defaults.

## Common Mistakes

- Using `utf8` instead of `utf8mb4` — causes silent data truncation with emoji and some Asian characters.
- Forgetting `USE <database>` before creating tables — triggers `No database selected` errors.
- Dropping a database without a backup — all data is gone with no recovery option.

## Next Step

Continue to [6-data-type.md](6-data-type.md).

