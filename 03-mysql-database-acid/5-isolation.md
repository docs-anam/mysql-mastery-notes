# Isolation

## What is Isolation?

**Isolation** guarantees that concurrently executing transactions do not interfere with each other. Each transaction behaves as if it is the only one running, even when hundreds of transactions are executing simultaneously.

> "Concurrent transactions produce results as if they ran serially, one after another."

---

## The Problem Isolation Solves

Consider two transactions running simultaneously:

```
Time  │  Transaction A (Transfer)         │  Transaction B (Report)
──────┼───────────────────────────────────┼───────────────────────────────
  1   │  START TRANSACTION                │
  2   │  UPDATE accounts ... (Alice -500) │
  3   │                                   │  START TRANSACTION
  4   │                                   │  SELECT SUM(balance)  ← sees Alice's new balance?
  5   │                                   │                          or the old balance?
  6   │  UPDATE accounts ... (Bob +500)   │
  7   │  COMMIT                           │
  8   │                                   │  COMMIT
```

What should Transaction B see at step 4?
- If it sees Alice's debited balance but not Bob's credited balance → the total appears $500 short (phantom money disappearance)
- This is a **concurrency anomaly** known as a *dirty read* or *non-repeatable read*

---

## Concurrency Anomalies

### 1. Dirty Read

Transaction B reads data that Transaction A has modified but **not yet committed**. If A rolls back, B has read data that never existed.

```
Tx A: UPDATE accounts SET balance = 0 WHERE id = 1;  ← not committed yet
Tx B: SELECT balance FROM accounts WHERE id = 1;      ← sees 0 (dirty read!)
Tx A: ROLLBACK;                                        ← A undoes the change
Tx B: uses 0 as Alice's balance — WRONG
```

### 2. Non-Repeatable Read

Transaction B reads the same row twice. Between the two reads, Transaction A commits a change to that row.

```
Tx B: SELECT balance FROM accounts WHERE id = 1;  → returns 5,000,000
Tx A: UPDATE accounts SET balance = 4,000,000 WHERE id = 1; COMMIT;
Tx B: SELECT balance FROM accounts WHERE id = 1;  → returns 4,000,000  ← different!
```

B gets two different values for the same row in the same transaction.

### 3. Phantom Read

Transaction B reads a set of rows matching a condition. Transaction A inserts a new row that matches the condition. Transaction B re-reads and finds a "phantom" new row.

```
Tx B: SELECT COUNT(*) FROM accounts WHERE balance > 1000000;  → 3
Tx A: INSERT INTO accounts (owner_name, balance) VALUES ('Eve', 2000000); COMMIT;
Tx B: SELECT COUNT(*) FROM accounts WHERE balance > 1000000;  → 4  ← phantom!
```

### Summary

| Anomaly | Cause |
|---------|-------|
| **Dirty Read** | Reading uncommitted changes from another transaction |
| **Non-Repeatable Read** | Same row reads differently within one transaction |
| **Phantom Read** | Same query returns different set of rows within one transaction |

---

## Isolation Levels

MySQL/InnoDB provides four isolation levels, offering a trade-off between **data accuracy** and **concurrency performance**.

```
More Isolation ←──────────────────────────────────────────→ More Concurrency
SERIALIZABLE   REPEATABLE READ   READ COMMITTED   READ UNCOMMITTED
```

### Comparison Table

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | InnoDB Locking |
|----------------|-----------|--------------------|--------------|----|
| `READ UNCOMMITTED` | ✅ Possible | ✅ Possible | ✅ Possible | Almost none |
| `READ COMMITTED` | ❌ Prevented | ✅ Possible | ✅ Possible | Row locks on write |
| `REPEATABLE READ` | ❌ Prevented | ❌ Prevented | ⚠️ Mostly prevented* | MVCC snapshot |
| `SERIALIZABLE` | ❌ Prevented | ❌ Prevented | ❌ Prevented | Range locks (slow) |

> *InnoDB `REPEATABLE READ` prevents phantom reads for snapshot reads (`SELECT`) via MVCC, but current reads (`SELECT ... FOR UPDATE`) can still see phantoms without gap locks.

---

## How InnoDB Prevents Anomalies: MVCC

InnoDB uses **MVCC (Multi-Version Concurrency Control)** to allow readers and writers to work concurrently without blocking each other.

```
InnoDB keeps multiple versions of each row:

Row (id=1)
├── Version 3: balance = 4,000,000  (committed at t=7)
├── Version 2: balance = 4,500,000  (committed at t=3)
└── Version 1: balance = 5,000,000  (original)

A reader transaction started at t=2 sees Version 1 (5,000,000)
regardless of what other transactions do after t=2.
```

This means:
- **Readers never block writers**
- **Writers never block readers**
- Each transaction gets a **consistent snapshot** of the database as it was when the transaction started

---

## Demonstrating Isolation Levels

### Check and Set the Current Level

```sql
-- View current isolation level
SHOW VARIABLES LIKE 'transaction_isolation';

-- Set for the current session
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- All four options:
-- SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Demonstrating REPEATABLE READ (InnoDB Default)

**Session A (open terminal 1):**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;
-- Returns: 4,000,000 (snapshot is taken here)
```

**Session B (open terminal 2):**

```sql
UPDATE accounts SET balance = 1000000 WHERE id = 1;
COMMIT;
-- Alice's balance is now 1,000,000 on disk
```

**Session A (continue):**

```sql
SELECT balance FROM accounts WHERE id = 1;
-- Still returns: 4,000,000  ← repeatable! MVCC serves the snapshot
COMMIT;
```

### Demonstrating READ COMMITTED

**Session A:**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;
-- Returns: 1,000,000 (current committed value)
```

**Session B:**

```sql
UPDATE accounts SET balance = 500000 WHERE id = 1;
COMMIT;
```

**Session A:**

```sql
SELECT balance FROM accounts WHERE id = 1;
-- Returns: 500,000  ← sees the new committed value (non-repeatable read is possible)
COMMIT;
```

---

## Locking and Isolation

Beyond MVCC snapshots, InnoDB uses **locks** for write isolation.

### Shared Lock (S-lock) — `FOR SHARE`

```sql
-- Tx A acquires a shared lock — others can read but not write
START TRANSACTION;
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- Tx B tries to update: BLOCKED until Tx A commits or rolls back
-- UPDATE accounts SET balance = 0 WHERE id = 1;  ← waits
```

### Exclusive Lock (X-lock) — `FOR UPDATE`

```sql
-- Tx A acquires an exclusive lock — nobody else can read or write
START TRANSACTION;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Tx B tries to read with lock: BLOCKED
-- SELECT * FROM accounts WHERE id = 1 FOR SHARE;  ← waits

-- Tx B tries to update: BLOCKED
-- UPDATE accounts SET balance = 0 WHERE id = 1;  ← waits
```

### Lock Compatibility Matrix

| | S-lock held | X-lock held |
|--|------------|------------|
| **Request S-lock** | ✅ Compatible | ❌ Blocked |
| **Request X-lock** | ❌ Blocked | ❌ Blocked |

### Gap Locks

InnoDB uses **gap locks** to prevent phantom reads in `REPEATABLE READ`. A gap lock prevents other transactions from inserting into a range of values.

```sql
-- Tx A locks all rows where balance BETWEEN 1000000 AND 5000000
-- (and the gap before and after them)
SELECT * FROM accounts WHERE balance BETWEEN 1000000 AND 5000000 FOR UPDATE;

-- Tx B tries to insert an account with balance 2000000: BLOCKED
-- INSERT INTO accounts (owner_name, balance) VALUES ('Eve', 2000000);  ← waits
```

---

## Deadlocks

A **deadlock** occurs when two transactions each hold a lock that the other needs.

```
Tx A holds lock on row 1, wants row 2
Tx B holds lock on row 2, wants row 1
→ InnoDB detects the cycle and kills the transaction with less work done
→ The killed transaction receives: ERROR 1213: Deadlock found
```

### Deadlock Example

```sql
-- Session A
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- locks row 1

-- Session B
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 2;  -- locks row 2

-- Session A (now wants row 2)
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- BLOCKED, waiting for B

-- Session B (now wants row 1)
UPDATE accounts SET balance = balance + 100 WHERE id = 1;  -- BLOCKED, waiting for A
-- Deadlock! InnoDB kills one transaction.
```

### How to Avoid Deadlocks

```sql
-- ALWAYS access rows in the same order across all transactions
-- If transfers always lock from_account first, then to_account:

START TRANSACTION;
-- Lock the lower ID first regardless of transfer direction
SELECT * FROM accounts WHERE id = LEAST(p_from, p_to) FOR UPDATE;
SELECT * FROM accounts WHERE id = GREATEST(p_from, p_to) FOR UPDATE;
-- Now update both
```

### Handling Deadlock Errors in Application Code

When a deadlock occurs, InnoDB automatically rolls back one transaction. The application must detect error 1213 and **retry**:

```python
# Pseudocode — retry pattern
max_retries = 3
for attempt in range(max_retries):
    try:
        execute_transfer(from_id, to_id, amount)
        break  # success
    except DeadlockError:
        if attempt == max_retries - 1:
            raise  # give up after max retries
        time.sleep(0.1 * (attempt + 1))  # exponential backoff
```

---

## Choosing the Right Isolation Level

| Scenario | Recommended Level |
|---------|------------------|
| **Most transactional workloads** (orders, payments) | `REPEATABLE READ` (default) |
| **Reporting / analytics queries** that need better concurrency | `READ COMMITTED` |
| **Financial audits** requiring absolute correctness | `SERIALIZABLE` |
| **High-throughput read-heavy feeds** (usually read-only replicas) | `READ COMMITTED` |
| **Never use** | `READ UNCOMMITTED` — provides false data |

---

## Best Practices

- Use `REPEATABLE READ` (the default) for transactional workloads. It offers the best balance of safety and concurrency.
- Always access tables and rows in a **consistent order** to avoid deadlocks.
- Keep transactions **short** — hold locks for as little time as possible.
- Always handle `ERROR 1213 (Deadlock)` in application code with a retry mechanism.
- Use `FOR UPDATE` only when you intend to modify the locked rows — unnecessary locking reduces concurrency.

## Common Mistakes

- Using `SERIALIZABLE` everywhere — it severely reduces concurrency and is rarely necessary.
- Not handling deadlocks in application code — the query fails silently and data is not written.
- Using `READ UNCOMMITTED` — you will process dirty data that may be rolled back.
- Opening a transaction, doing lots of work, then running a `SELECT ... FOR UPDATE` — you hold locks way too long.

## Next Step

Continue to [6-durability.md](6-durability.md) for the final ACID property — Durability.