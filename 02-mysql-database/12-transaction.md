# Transactions, Locking, Users, and Backup

## What is a Transaction?

A **transaction** is a group of SQL statements that are executed as a single unit of work. Either **all statements succeed** (commit) or **all are rolled back** (undo) as if they never happened.

```
Without transaction:
  Step 1: Debit account A  ✅
  Step 2: Credit account B ❌ FAILS  → Account A already debited! Data corrupted.

With transaction:
  Step 1: Debit account A  ✅
  Step 2: Credit account B ❌ FAILS  → ROLLBACK → Both steps undone. Safe.
```

---

## ACID Properties

Every transaction in InnoDB is guaranteed to satisfy **ACID**:

| Property | Meaning |
|----------|---------|
| **A**tomicity | All statements in the transaction succeed or all are rolled back |
| **C**onsistency | Data moves from one valid state to another valid state — constraints are always satisfied |
| **I**solation | Concurrent transactions don't interfere with each other |
| **D**urability | Once committed, changes survive crashes and power failures |

---

## Transaction Commands

```sql
START TRANSACTION;   -- Begin a transaction block

COMMIT;              -- Save all changes permanently

ROLLBACK;            -- Undo all changes made since START TRANSACTION

SAVEPOINT name;      -- Mark a named checkpoint inside the transaction

ROLLBACK TO name;    -- Undo to the savepoint (without rolling back the full transaction)

RELEASE SAVEPOINT name;  -- Remove a savepoint (optional)
```

---

## Basic Transaction Example

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100000 WHERE id = 1;
UPDATE accounts SET balance = balance + 100000 WHERE id = 2;

-- If both updates succeed:
COMMIT;

-- If anything fails, run this instead:
-- ROLLBACK;
```

---

## Savepoints

Savepoints allow **partial rollbacks** within a transaction.

```sql
START TRANSACTION;

INSERT INTO orders (customer_id, total_amount) VALUES (5, 500000);
SAVEPOINT after_order;       -- mark this point

INSERT INTO order_items (order_id, product_id, quantity) VALUES (LAST_INSERT_ID(), 10, 2);
-- Suppose this fails:
ROLLBACK TO after_order;     -- undo the order_item insert only

-- The order insert is still pending
COMMIT;                      -- save just the order
```

---

## Autocommit Mode

By default, MySQL runs in **autocommit** mode — every statement is automatically committed.

```sql
-- Check current autocommit setting
SHOW VARIABLES LIKE 'autocommit';

-- Disable autocommit for the session
SET autocommit = 0;

-- Now statements aren't committed until you explicitly COMMIT
INSERT INTO orders (...) VALUES (...);
COMMIT;  -- must call explicitly

-- Re-enable
SET autocommit = 1;
```

> `START TRANSACTION` temporarily overrides autocommit for the duration of that transaction block.

---

## Locking

MySQL uses **locks** to coordinate concurrent access and prevent race conditions.

### Row-Level Locking with FOR UPDATE

```sql
START TRANSACTION;

-- Lock the selected row so no other transaction can modify it
SELECT * FROM orders WHERE id = 100 FOR UPDATE;

-- Now safely update knowing no concurrent modification can occur
UPDATE orders SET status = 'confirmed' WHERE id = 100;

COMMIT;
```

### Shared Lock (FOR SHARE)

```sql
START TRANSACTION;

-- Allow other transactions to read but not modify
SELECT * FROM orders WHERE id = 100 FOR SHARE;

-- ... read data, no changes ...
COMMIT;
```

| Lock Type | Acquired With | Allows Concurrent Reads | Allows Concurrent Writes |
|-----------|--------------|------------------------|--------------------------|
| Exclusive (write) | `FOR UPDATE` | ❌ | ❌ |
| Shared (read) | `FOR SHARE` | ✅ | ❌ |

### Deadlocks

A **deadlock** occurs when two transactions each hold a lock the other needs.

```
Transaction A holds lock on row 1, waits for row 2
Transaction B holds lock on row 2, waits for row 1
→ Neither can proceed → MySQL detects this and kills one transaction
```

**Strategies to avoid deadlocks:**
- Access tables and rows in a **consistent order** across all transactions.
- Keep transactions **short** — acquire and release locks quickly.
- Use **lower isolation levels** when strict isolation isn't required.

---

## Transaction Isolation Levels

Isolation levels control how much one transaction can "see" the in-progress work of another.

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use Case |
|-------|-----------|--------------------|--------------|----|
| `READ UNCOMMITTED` | ✅ Yes | ✅ Yes | ✅ Yes | Rarely used — dangerous |
| `READ COMMITTED` | ❌ No | ✅ Yes | ✅ Yes | Reporting, analytics |
| `REPEATABLE READ` | ❌ No | ❌ No | ✅ Yes | **InnoDB default** |
| `SERIALIZABLE` | ❌ No | ❌ No | ❌ No | Strictest — high locking overhead |

```sql
-- View current isolation level
SHOW VARIABLES LIKE 'transaction_isolation';

-- Change for the current session
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Change for next transaction only
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

> **Default `REPEATABLE READ`** is appropriate for most applications. Switch to `READ COMMITTED` when you need better concurrency for reporting queries.

---

## Best Practices

- Keep transactions **short** — long transactions hold locks that block other operations.
- Always wrap multi-step data modifications (debit/credit, insert parent+children) in a transaction.
- Use `REPEATABLE READ` (the InnoDB default) for most workloads; switch to `READ COMMITTED` only for reporting queries that need better concurrency.
- Avoid nested `FOR UPDATE` on multiple tables in different orders — this is the most common cause of deadlocks.

## Common Mistakes

- Running long transactions that hold locks and block other users for minutes.
- Not handling the case where a transaction is automatically rolled back due to a deadlock — the caller must retry.
- Using `ROLLBACK TO SAVEPOINT` and assuming the full transaction is undone — the rest of the transaction is still open.
- Committing without verifying that all steps in a multi-step operation succeeded.

## Next Step

Continue to [13-user-management.md](13-user-management.md) to learn how to manage users and permissions safely.

