# ACID Properties in MySQL

## Overview

This module takes a deep dive into **ACID** — the four guarantees that make MySQL (and every InnoDB-based database) safe and reliable for real-world applications. You already used transactions in the previous module; now you'll understand *why* they work the way they do and how to design code that never leaves your database in a broken state.

## What is ACID?

**ACID** is an acronym for four properties that every database transaction must satisfy:

```
┌───────────────────────────────────────────────────────────────┐
│                          ACID                                 │
│                                                               │
│  A — Atomicity     All or nothing. No partial transactions.   │
│  C — Consistency   Data always moves between valid states.    │
│  I — Isolation     Transactions don't see each other's drafts.│
│  D — Durability    Committed data survives crashes.           │
└───────────────────────────────────────────────────────────────┘
```

These four properties work together to ensure that concurrent database operations on shared data are **safe, correct, and recoverable**.

## Why ACID Matters

### The Classic Problem — Bank Transfer

```
Transfer $100 from Account A to Account B:

Step 1: Debit  Account A  (balance: 500 → 400) ✅
Step 2: Credit Account B  (balance: 200 → ???)

What if the server crashes between Step 1 and Step 2?
```

Without ACID:
- Account A has lost $100
- Account B never received it
- **$100 has vanished from reality** — data is corrupt

With ACID (InnoDB):
- The incomplete transaction is rolled back automatically on recovery
- Both accounts are restored to their original balances
- No money is ever lost

## Learning Path

| # | Topic | What You Learn |
|---|-------|----------------|
| 1 | [Project Setup](2-setup-project.md) | The sample bank schema used throughout this module |
| 2 | [Atomicity](3-atomicity.md) | All-or-nothing execution; `ROLLBACK` on failure |
| 3 | [Consistency](4-consistency.md) | Constraints, triggers, and valid state transitions |
| 4 | [Isolation](5-isolation.md) | Concurrency anomalies and isolation levels |
| 5 | [Durability](6-durability.md) | Write-ahead log, crash recovery, and InnoDB internals |

## ACID in InnoDB

MySQL's default storage engine, **InnoDB**, provides full ACID compliance:

```
InnoDB Mechanisms
├── Atomicity   →  Undo log (rollback segments)
├── Consistency →  Constraint enforcement + triggers
├── Isolation   →  MVCC (Multi-Version Concurrency Control) + locks
└── Durability  →  Redo log (Write-Ahead Log) + fsync
```

## Prerequisites

Before this module, you should be comfortable with:
- Writing basic `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- Using `START TRANSACTION`, `COMMIT`, `ROLLBACK`
- Table relationships and foreign keys

All of these are covered in [02-mysql-database](../02-mysql-database/1-intro.md).

## Next Step

Start with **[Project Setup](2-setup-project.md)** to create the sample schema used in all examples throughout this module.