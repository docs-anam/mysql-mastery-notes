# Introduction to MySQL

## Overview

This module builds on the database fundamentals in section 01 and moves into **practical MySQL usage**. You will learn how to install MySQL, model schemas, write SQL queries, manage data integrity, optimize performance, and apply production-ready practices.

## What is MySQL?

**MySQL** is the world's most popular open-source relational database management system (RDBMS). It is:

- **Fast** — optimized storage engine (InnoDB) handles millions of rows efficiently
- **Reliable** — full ACID transaction support ensures data consistency
- **Widely supported** — used by companies like Facebook, Twitter, Airbnb, and Wikipedia
- **Standard SQL** — follows SQL standards with useful MySQL-specific extensions
- **Free & open source** — community edition under GPL license

```
┌─────────────────────────────────────────────┐
│               MySQL Ecosystem               │
│                                             │
│  ┌─────────────┐    ┌──────────────────┐    │
│  │  Your App   │───▶│  MySQL Server    │    │
│  │  (backend)  │    │  (mysqld daemon) │    │
│  └─────────────┘    └────────┬─────────┘    │
│                              │              │
│                   ┌──────────▼──────────┐   │
│                   │  InnoDB Storage     │   │
│                   │  Engine             │   │
│                   │  – ACID compliance  │   │
│                   │  – Row-level locks  │   │
│                   │  – Foreign keys     │   │
│                   └─────────────────────┘   │
└─────────────────────────────────────────────┘
```

## Learning Path

| # | Topic | What You Learn |
|---|-------|----------------|
| 1 | [MySQL Client and Connecting](2-mysql-client.md) | MySQL CLI, session commands, server architecture |
| 2 | [Introduction to SQL](3-intro-sql.md) | SQL categories (DDL/DML/DQL/TCL/DCL), query anatomy |
| 3 | [Installing MySQL](4-install-mysql.md) | Setup on macOS, Linux, and Docker |
| 4 | [Working with Databases](5-database.md) | CREATE, USE, DROP, character sets, collations |
| 5 | [Data Types](6-data-type.md) | Numeric, string, date-time, JSON, ENUM |
| 6 | [Tables and CRUD](7-table.md) | CREATE TABLE, INSERT, SELECT, UPDATE, DELETE |
| 7 | [Filtering, Sorting, Pagination](8-filtering-sorting-pagination.md) | WHERE, ORDER BY, LIMIT/OFFSET, aliases |
| 8 | [Functions and Grouping](9-functions-and-grouping.md) | Built-in functions, GROUP BY, HAVING |
| 9 | [Constraints, Indexes, and Search](10-constraint.md) | Data integrity, indexes, full-text search |
| 10 | [Relationships and Joins](11-table-relationship.md) | JOINs, subqueries, relationship patterns |
| 11 | [Transactions and Locking](12-transaction.md) | ACID, START/COMMIT/ROLLBACK, isolation levels |
| 12 | [User and Permission Management](13-user-management.md) | CREATE USER, GRANT, REVOKE, least privilege |
| 13 | [Backup and Restore](14-backup-restore.md) | mysqldump, restore, binary logs, backup strategy |

## Why Learn MySQL?

Understanding MySQL deeply is **essential** because:

- ✅ It is used in production at massive scale worldwide
- ✅ Correct schema design prevents costly data migrations later
- ✅ Proper indexing can make queries 100x faster
- ✅ Understanding transactions protects against data corruption
- ✅ Most backend frameworks (Laravel, Django, Rails, Spring) use MySQL as their primary database

## Prerequisites

Before starting this module, make sure you understand:

- **Entities and attributes** — what a table and column represent
- **Primary and foreign keys** — how tables relate to each other
- **Normalization** — basic 1NF, 2NF, 3NF concepts
- **ERDs** — how to read entity-relationship diagrams

All of these are covered in the [01-database](../01-database/1-intro.md) module.

## Key Concepts to Master

By the end of this module you should be able to:

- **Design** a normalized relational schema from requirements
- **Write** correct SELECT, INSERT, UPDATE, and DELETE statements
- **Use** JOINs to combine data across multiple tables
- **Apply** constraints and indexes to ensure integrity and performance
- **Manage** transactions to protect multi-step operations
- **Create** users and manage permissions securely
- **Back up and restore** a MySQL database

## Next Step

Start with **[MySQL Client and Connecting](2-mysql-client.md)** to learn how to connect to and navigate a MySQL server.
