# Project Setup — Banking Schema

## Why a Banking Schema?

The classic ACID demonstration is a **bank transfer**: money must move from one account to another atomically — it can never appear in both accounts simultaneously, nor in neither. It is the simplest scenario that makes all four ACID properties visible and testable.

This schema is used in every subsequent chapter.

---

## Schema Design

```
┌─────────────────────┐         ┌─────────────────────────────┐
│      accounts       │         │        transfers            │
├─────────────────────┤         ├─────────────────────────────┤
│ id (PK)             │         │ id (PK)                     │
│ owner_name          │ ────┐   │ from_account_id (FK)        │
│ balance             │     └──▶│ to_account_id (FK)          │
│ created_at          │         │ amount                      │
└─────────────────────┘         │ note                        │
                                │ transferred_at              │
                                └─────────────────────────────┘
```

---

## Create the Database and Tables

```sql
-- 1. Create the database
CREATE DATABASE IF NOT EXISTS bank
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE bank;

-- 2. Accounts table: stores each customer's current balance
CREATE TABLE accounts (
  id         BIGINT         PRIMARY KEY AUTO_INCREMENT,
  owner_name VARCHAR(120)   NOT NULL,
  balance    DECIMAL(15, 2) NOT NULL DEFAULT 0.00,
  created_at TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT chk_balance_non_negative CHECK (balance >= 0)
);

-- 3. Transfers table: immutable audit log of every movement
CREATE TABLE transfers (
  id              BIGINT         PRIMARY KEY AUTO_INCREMENT,
  from_account_id BIGINT         NOT NULL,
  to_account_id   BIGINT         NOT NULL,
  amount          DECIMAL(15, 2) NOT NULL,
  note            VARCHAR(255),
  transferred_at  TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,

  CONSTRAINT chk_transfer_amount_positive CHECK (amount > 0),
  CONSTRAINT chk_transfer_different_accounts
    CHECK (from_account_id != to_account_id),

  FOREIGN KEY (from_account_id) REFERENCES accounts(id) ON DELETE RESTRICT,
  FOREIGN KEY (to_account_id)   REFERENCES accounts(id) ON DELETE RESTRICT
);

-- 4. Index for fast lookups by account
CREATE INDEX idx_transfers_from ON transfers(from_account_id, transferred_at);
CREATE INDEX idx_transfers_to   ON transfers(to_account_id,   transferred_at);
```

---

## Seed Data

```sql
-- Insert four accounts with starting balances
INSERT INTO accounts (owner_name, balance) VALUES
  ('Alice',   5000000.00),
  ('Bob',     3000000.00),
  ('Charlie', 1500000.00),
  ('Diana',   8000000.00);

-- Verify
SELECT id, owner_name, balance FROM accounts;
```

Expected result:

```
+----+------------+------------+
| id | owner_name | balance    |
+----+------------+------------+
|  1 | Alice      | 5000000.00 |
|  2 | Bob        | 3000000.00 |
|  3 | Charlie    | 1500000.00 |
|  4 | Diana      | 8000000.00 |
+----+------------+------------+
```

---

## Helper View — Account Summary

```sql
CREATE VIEW account_summary AS
SELECT
  a.id,
  a.owner_name,
  a.balance,
  (SELECT COALESCE(SUM(t.amount), 0)
   FROM transfers t WHERE t.from_account_id = a.id) AS total_sent,
  (SELECT COALESCE(SUM(t.amount), 0)
   FROM transfers t WHERE t.to_account_id = a.id)   AS total_received
FROM accounts a;
```

```sql
SELECT * FROM account_summary;
```

---

## Transfer Stored Procedure

Throughout this module we will call `transfer_funds` to perform a transfer. This stored procedure will be built up incrementally as each ACID property is introduced.

```sql
DELIMITER $$

CREATE PROCEDURE transfer_funds(
  IN p_from  BIGINT,
  IN p_to    BIGINT,
  IN p_amount DECIMAL(15, 2),
  IN p_note  VARCHAR(255)
)
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    ROLLBACK;
    RESIGNAL;
  END;

  START TRANSACTION;

    -- Debit the sender
    UPDATE accounts
    SET balance = balance - p_amount
    WHERE id = p_from;

    -- Credit the recipient
    UPDATE accounts
    SET balance = balance + p_amount
    WHERE id = p_to;

    -- Record the transfer
    INSERT INTO transfers (from_account_id, to_account_id, amount, note)
    VALUES (p_from, p_to, p_amount, p_note);

  COMMIT;
END$$

DELIMITER ;
```

---

## Verify the Setup

```sql
-- Tables created correctly
SHOW TABLES;

-- Schema for each table
DESCRIBE accounts;
DESCRIBE transfers;

-- Seed data present
SELECT COUNT(*) FROM accounts;  -- should be 4

-- Check constraints are in place
SHOW CREATE TABLE accounts;
SHOW CREATE TABLE transfers;
```

---

## Best Practices

- Always keep a **transfers** (audit log) table separate from the **accounts** balance table — never modify balances without an audit trail.
- Use `DECIMAL(15, 2)` for money — never `FLOAT` or `DOUBLE`.
- Add `CHECK (balance >= 0)` to prevent negative balances at the database level — not just in application code.

## Next Step

Continue to [3-atomicity.md](3-atomicity.md) for the first ACID property — Atomicity.