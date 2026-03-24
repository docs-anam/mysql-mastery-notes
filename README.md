# MySQL Mastery Notes

A comprehensive, structured learning resource for mastering MySQL from complete beginner to advanced user. This repository contains detailed guides, practical code examples, and best practices for using MySQL effectively in production environments.

## 📚 What's Inside

This repository is organized into progressive learning folders, each building on previous concepts:

### 1. **Database Fundamentals** (`01-database/`) - Database Concepts & Theory
Learn the foundational concepts of databases and relational database design.

- **[1-intro.md](01-database/1-intro.md)** - What is a database? RDBMS concepts and MySQL overview

### 2. **MySQL Basics** (`02-mysql-database/`) - Essential Operations & Commands
Learn the fundamentals of MySQL and essential commands for daily use.

- **[1-intro.md](02-mysql-database/1-intro.md)** - Module overview and recommended learning sequence
- **[2-mysql-client.md](02-mysql-database/2-mysql-client.md)** - MySQL CLI, connection flags, and server architecture
- **[3-intro-sql.md](02-mysql-database/3-intro-sql.md)** - SQL basics and command categories
- **[4-install-mysql.md](02-mysql-database/4-install-mysql.md)** - Installation and initial setup
- **[5-database.md](02-mysql-database/5-database.md)** - Creating and managing databases
- **[6-data-type.md](02-mysql-database/6-data-type.md)** - Data types consolidated guide
- **[7-table.md](02-mysql-database/7-table.md)** - Table design and CRUD consolidated guide
- **[8-filtering-sorting-pagination.md](02-mysql-database/8-filtering-sorting-pagination.md)** - WHERE operators, ORDER BY, and LIMIT/OFFSET
- **[9-functions-and-grouping.md](02-mysql-database/9-functions-and-grouping.md)** - SQL functions, GROUP BY, and HAVING
- **[10-constraint.md](02-mysql-database/10-constraint.md)** - Constraints, indexing, and full-text search
- **[11-table-relationship.md](02-mysql-database/11-table-relationship.md)** - Relationships, JOINs, subqueries, and set operators
- **[12-transaction.md](02-mysql-database/12-transaction.md)** - Transactions, isolation levels, and locking
- **[13-user-management.md](02-mysql-database/13-user-management.md)** - User accounts, GRANT/REVOKE, and least-privilege design
- **[14-backup-restore.md](02-mysql-database/14-backup-restore.md)** - mysqldump, binary log, and point-in-time recovery

### 3. **ACID Properties & Transactions** (`03-mysql-database-acid/`) - Data Integrity & Consistency
Comprehensive guides on transactions, ACID properties, and data consistency.

- **[1-intro.md](03-mysql-database-acid/1-intro.md)** - ACID overview, learning path, and InnoDB mechanisms
- **[2-setup-project.md](03-mysql-database-acid/2-setup-project.md)** - Banking schema project setup with seed data
- **[3-atomicity.md](03-mysql-database-acid/3-atomicity.md)** - Atomicity: all-or-nothing transactions and undo log
- **[4-consistency.md](03-mysql-database-acid/4-consistency.md)** - Consistency: constraints, triggers, and valid state
- **[5-isolation.md](03-mysql-database-acid/5-isolation.md)** - Isolation: concurrency anomalies, levels, and MVCC
- **[6-durability.md](03-mysql-database-acid/6-durability.md)** - Durability: redo log, fsync, and crash recovery

### 4. **Database Design: Notification System** (`04-database-design-notification/`) - Real-World Case Study
Practical example of designing a notification system database.

- **[1-intro.md](04-database-design-notification/1-intro.md)** - Overview, architecture, and key design challenges
- **[2-requirements.md](04-database-design-notification/2-requirements.md)** - Functional and non-functional requirements, use cases
- **[3-setup.md](04-database-design-notification/3-setup.md)** - Database and users table setup
- **[4-categories.md](04-database-design-notification/4-categories.md)** - Notification categories and routing logic
- **[5-preferences.md](04-database-design-notification/5-preferences.md)** - User notification preferences and category opt-in/out
- **[6-inbox.md](04-database-design-notification/6-inbox.md)** - Notifications and user_notifications (inbox) tables
- **[7-read-status.md](04-database-design-notification/7-read-status.md)** - Read/unread status tracking with read_at timestamp
- **[8-counter.md](04-database-design-notification/8-counter.md)** - Unread badge counter with triggers and reconciliation

### 5. **Database Design: Multi-Language Support** (`05-database-design-multi-languages/`) - Real-World Case Study
Designing databases for multi-language and internationalization support.

- *Content to be added*

### 6. **Database Design: Tokopedia Order History** (`06-database-design-tokopedia-order-history/`) - Real-World Case Study
Advanced database design patterns based on an e-commerce order history system.

- *Content to be added*

## 🚀 Features

Each guide includes:

- **📖 Detailed Explanations** - Comprehensive coverage with examples
- **💻 SQL Code Examples** - Practical queries and schema definitions
- **🎯 Real-World Use Cases** - Production scenarios with working implementations
- **✅ Best Practices** - What to do and what to avoid
- **⚠️ Common Mistakes** - Pitfalls and how to prevent them
- **🔗 Next Steps** - References to related topics

## 💡 Prerequisites

- Basic command-line comfort
- Fundamental programming knowledge
- MySQL installed locally or access to a MySQL instance

## 🔧 Quick Start

1. Start with **Database Fundamentals** in `01-database/`.
2. Continue with **MySQL Basics** in `02-mysql-database/`.
3. Practice every query in a local MySQL instance.
4. Build a small schema (users, products, orders) and iterate after each chapter.