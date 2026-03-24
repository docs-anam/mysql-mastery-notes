# Atomicity

## What is Atomicity?

**Atomicity** guarantees that a transaction is treated as a **single, indivisible unit**. Either every statement in the transaction succeeds and is committed, or every statement is rolled back — as if the transaction never happened.

> "All or nothing."

---

## The Problem Atomicity Solves

Without atomicity, a partial failure mid-transaction corrupts data permanently:

```
Transfer $500,000 from Alice to Bob:

Step 1: UPDATE accounts SET balance = balance - 500000 WHERE id = 1; ✅ Alice: 5,000,000 → 4,500,000
Step 2: UPDATE accounts SET balance = balance + 500000 WHERE id = 2; ❌ SERVER CRASH

Result without atomicity:
  Alice: 4,500,000  (debited — money is gone)
  Bob:   3,000,000  (never credited)
  500,000 has disappeared from the system
```

With atomicity, InnoDB's **undo log** records the before-image of every changed row. On crash recovery, the incomplete transaction is rolled back automatically.

---

## How InnoDB Implements Atomicity

```
START TRANSACTION
        │
    ┌───▼───────────────────────────────────────┐
    │  For each row modified:                   │
    │  1. Write before-image to UNDO LOG        │
    │  2. Apply the change to the buffer pool   │
    └───┬───────────────────────────────────────┘
        │
   COMMIT? ──────YES──────▶  Mark transaction as committed
        │                     Undo log entries become purgeable
        │
       NO (error or ROLLBACK)
        │
        ▼
   Read undo log entries in reverse → undo every change → data is restored
```

The **undo log** is stored inside the InnoDB tablespace. It is the foundation of atomicity.

---

## Demonstrating Atomicity

### Setup: verify starting balances

```sql
SELECT id, owner_name, balance FROM accounts;
-- Alice: 5,000,000 | Bob: 3,000,000
```

### Test 1 — Successful Transaction (COMMIT)

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500000 WHERE id = 1; -- Alice
UPDATE accounts SET balance = balance + 500000 WHERE id = 2; -- Bob

INSERT INTO transfers (from_account_id, to_account_id, amount, note)
VALUES (1, 2, 500000, 'Test transfer');

COMMIT;

-- Verify: balances changed, transfer recorded
SELECT id, owner_name, balance FROM accounts;
SELECT * FROM transfers;
```

Expected result: Alice = 4,500,000 | Bob = 3,500,000

---

### Test 2 — Explicit ROLLBACK

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 1000000 WHERE id = 1; -- Alice debited

-- Simulate a business rule failure in application code
SELECT balance FROM accounts WHERE id = 1;
-- Suppose the application checks and finds the amount is too large
ROLLBACK;

-- Verify: Alice's balance is unchanged
SELECT id, owner_name, balance FROM accounts WHERE id = 1;
-- Alice: 4,500,000 (unchanged from previous test)
```

---

### Test 3 — Constraint Violation Triggers Automatic Rollback

The `CHECK (balance >= 0)` constraint causes the transaction to fail if balance would go negative.

```sql
START TRANSACTION;

-- Try to overdraft Alice (current balance: 4,500,000)
UPDATE accounts SET balance = balance - 9999999 WHERE id = 1;
-- This attempt violates CHECK (balance >= 0)
-- MySQL raises an error → the UPDATE fails

-- The transaction is now in an error state
-- Any further statements are rejected until ROLLBACK or COMMIT
ROLLBACK;

-- Alice's balance remains 4,500,000
SELECT id, owner_name, balance FROM accounts WHERE id = 1;
```

---

### Test 4 — Using SAVEPOINTS for Partial Rollback

Savepoints let you define checkpoints **within** a transaction. You can roll back to a savepoint without abandoning the whole transaction.

```sql
START TRANSACTION;

-- Transfer 1: Alice → Bob (OK)
UPDATE accounts SET balance = balance - 200000 WHERE id = 1;
UPDATE accounts SET balance = balance + 200000 WHERE id = 2;
INSERT INTO transfers (from_account_id, to_account_id, amount, note)
VALUES (1, 2, 200000, 'Payment for services');

SAVEPOINT after_first_transfer;

-- Transfer 2: Alice → Charlie (attempt — suppose this fails validation)
UPDATE accounts SET balance = balance - 999999999 WHERE id = 1;
-- Balance would go negative → error

ROLLBACK TO after_first_transfer;  -- undo only the second transfer

-- First transfer is still pending
COMMIT;  -- saves only the first transfer

SELECT id, owner_name, balance FROM accounts;
SELECT * FROM transfers;
```

---

## Autocommit and Atomicity

When autocommit is ON (the default), every single statement runs in its own implicit transaction:

```sql
-- autocommit = ON (default)
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ↑ This is automatically wrapped in: START TRANSACTION; ... COMMIT;
-- It either fully succeeds or fully fails — it never partially applies.
```

This means even without an explicit `START TRANSACTION`, single statements are atomic. The danger arises only when you need **multiple statements** to be atomic together — that's when you must use `START TRANSACTION` explicitly.

---

## Error Handling Pattern in Application Code

In application code, always wrap multi-step operations in try/catch with explicit rollback:

```sql
-- Pattern for any language (pseudocode mapped to SQL)
START TRANSACTION;

  UPDATE accounts SET balance = balance - p_amount WHERE id = p_from;
  -- Check: did balance go negative? (application check)
  UPDATE accounts SET balance = balance + p_amount WHERE id = p_to;
  INSERT INTO transfers (...) VALUES (...);

COMMIT;
-- On any exception caught by the application:
-- ROLLBACK;
```

The stored procedure created in [2-setup-project.md](2-setup-project.md) implements this with a `DECLARE EXIT HANDLER FOR SQLEXCEPTION` that auto-rolls back on any error.

---

## Best Practices

- Always use `START TRANSACTION` when two or more statements must succeed or fail together.
- Set up `DECLARE EXIT HANDLER FOR SQLEXCEPTION BEGIN ROLLBACK; RESIGNAL; END;` in stored procedures.
- Never rely on catching only specific error codes — use a general `SQLEXCEPTION` handler as a safety net.
- After a `ROLLBACK`, verify the transaction is closed before issuing new statements.

## Common Mistakes

- Running two `UPDATE` statements without a transaction and assuming they will always both succeed.
- Catching an exception but forgetting to `ROLLBACK` — the connection remains in a broken transaction state.
- Using `TRUNCATE` inside a transaction — `TRUNCATE` is DDL and implicitly commits, breaking atomicity.
- Relying only on application-level validation without `CHECK` constraints — a bug or direct DB access bypasses the app.

## Next Step

Continue to [4-consistency.md](4-consistency.md) for the second ACID property — Consistency.