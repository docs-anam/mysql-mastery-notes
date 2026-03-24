# Consistency

## What is Consistency?

**Consistency** guarantees that a transaction always moves the database from one **valid state** to another **valid state**. It can never leave the database in a state that violates its own rules.

> "Every transaction leaves the database in a legal state."

---

## What Defines a "Valid State"?

A valid state is one where **all defined rules are satisfied**:

```
Rules that define consistency:
├── Column constraints   (NOT NULL, UNIQUE, CHECK, DEFAULT)
├── Table constraints    (PRIMARY KEY, FOREIGN KEY)
├── Domain rules         (balance >= 0, age BETWEEN 0 AND 150)
├── Referential rules    (every order must reference a valid customer)
└── Business rules       (total of all debits = total of all credits)
```

Consistency is the only ACID property that is **partly the database's responsibility and partly yours** as the schema designer. The database enforces what you declare; you are responsible for declaring the right rules.

---

## Constraints as Consistency Enforcers

### NOT NULL — Required Fields

```sql
-- owner_name cannot be empty
CREATE TABLE accounts (
  id         BIGINT       PRIMARY KEY AUTO_INCREMENT,
  owner_name VARCHAR(120) NOT NULL,   -- enforces consistency: every account has a name
  balance    DECIMAL(15,2) NOT NULL DEFAULT 0.00
);

-- This will be rejected:
INSERT INTO accounts (balance) VALUES (1000.00);
-- ERROR 1364: Field 'owner_name' doesn't have a default value
```

### CHECK — Domain Rules

```sql
-- Balance must never be negative — a fundamental bank invariant
ALTER TABLE accounts
  ADD CONSTRAINT chk_balance_non_negative CHECK (balance >= 0);

-- Transfer amount must be positive
ALTER TABLE transfers
  ADD CONSTRAINT chk_transfer_positive CHECK (amount > 0);

-- An account cannot transfer to itself
ALTER TABLE transfers
  ADD CONSTRAINT chk_no_self_transfer
  CHECK (from_account_id != to_account_id);
```

### FOREIGN KEY — Referential Integrity

```sql
-- Every transfer must reference accounts that actually exist
ALTER TABLE transfers
  ADD FOREIGN KEY (from_account_id) REFERENCES accounts(id) ON DELETE RESTRICT,
  ADD FOREIGN KEY (to_account_id)   REFERENCES accounts(id) ON DELETE RESTRICT;

-- This is rejected: account 999 does not exist
INSERT INTO transfers (from_account_id, to_account_id, amount)
VALUES (1, 999, 50000);
-- ERROR 1452: Cannot add or update a child row: a foreign key constraint fails
```

### UNIQUE — No Duplicate Business Keys

```sql
CREATE TABLE customers (
  id    BIGINT       PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(120) NOT NULL UNIQUE  -- one email = one customer
);

-- This second insert is rejected:
INSERT INTO customers (email) VALUES ('alice@example.com');
INSERT INTO customers (email) VALUES ('alice@example.com');
-- ERROR 1062: Duplicate entry 'alice@example.com' for key 'email'
```

---

## Demonstrating Consistency

### Test 1 — Constraint Blocks Invalid Transfer Amount

```sql
START TRANSACTION;

-- Attempt to transfer 0 (violates CHECK amount > 0)
INSERT INTO transfers (from_account_id, to_account_id, amount, note)
VALUES (1, 2, 0, 'Zero transfer');
-- ERROR: Check constraint 'chk_transfer_positive' is violated.

ROLLBACK;
-- Database remains in a consistent state — no records inserted
```

### Test 2 — Constraint Blocks Overdraft

```sql
SELECT balance FROM accounts WHERE id = 1;
-- Alice: 4,300,000 (from previous tests)

START TRANSACTION;

UPDATE accounts SET balance = balance - 9999999 WHERE id = 1;
-- ERROR: Check constraint 'chk_balance_non_negative' is violated.

ROLLBACK;

-- Alice's balance is still 4,300,000
SELECT balance FROM accounts WHERE id = 1;
```

### Test 3 — Consistency Across Tables

A valid transfer must:
1. Reduce the sender's balance by exactly `amount`
2. Increase the recipient's balance by exactly `amount`
3. Record an entry in `transfers`

All three together form a **consistent state transition**:

```sql
-- Before: Alice = 4,300,000 | Bob = 3,700,000
--         Total = 8,000,000

START TRANSACTION;

UPDATE accounts SET balance = balance - 300000 WHERE id = 1;
UPDATE accounts SET balance = balance + 300000 WHERE id = 2;
INSERT INTO transfers (from_account_id, to_account_id, amount, note)
VALUES (1, 2, 300000, 'Rent payment');

COMMIT;

-- After: Alice = 4,000,000 | Bob = 4,000,000
--        Total = 8,000,000 (UNCHANGED — consistent!)
SELECT owner_name, balance FROM accounts WHERE id IN (1, 2);
```

```sql
-- Business-level invariant: total money in the system never changes
SELECT SUM(balance) AS total_money FROM accounts;
-- Always returns the same value regardless of how many transfers occur
```

---

## Triggers — Enforcing Complex Business Rules

Some rules are too complex to express as a simple `CHECK` constraint. **Triggers** let you enforce them automatically.

### Example: Prevent Transfers When Sender's Account is Inactive

```sql
-- Add an active flag
ALTER TABLE accounts ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT 1;

DELIMITER $$

CREATE TRIGGER trg_check_active_before_debit
BEFORE UPDATE ON accounts
FOR EACH ROW
BEGIN
  -- If balance is being reduced and the account is inactive, reject
  IF NEW.balance < OLD.balance AND OLD.is_active = 0 THEN
    SIGNAL SQLSTATE '45000'
      SET MESSAGE_TEXT = 'Cannot debit an inactive account';
  END IF;
END$$

DELIMITER ;
```

```sql
-- Test the trigger
UPDATE accounts SET is_active = 0 WHERE id = 3; -- deactivate Charlie

START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 3; -- try to debit Charlie
-- ERROR 1644: Cannot debit an inactive account
ROLLBACK;
```

### Example: Enforce Total Balance Invariant

```sql
DELIMITER $$

CREATE TRIGGER trg_enforce_transfer_integrity
AFTER INSERT ON transfers
FOR EACH ROW
BEGIN
  DECLARE v_balance DECIMAL(15,2);

  SELECT balance INTO v_balance
  FROM accounts WHERE id = NEW.from_account_id;

  IF v_balance < 0 THEN
    SIGNAL SQLSTATE '45000'
      SET MESSAGE_TEXT = 'Consistency violation: negative balance detected after transfer';
  END IF;
END$$

DELIMITER ;
```

---

## Consistency vs Application-Level Validation

| Enforcement Layer | Example | Can be bypassed? |
|------------------|---------|-----------------|
| Database constraint (`CHECK`, `FK`, `UNIQUE`) | `balance >= 0` in schema | ❌ No — always enforced |
| Database trigger | Active account check | ❌ No — always enforced |
| Application code | `if (amount <= 0) throw Error` | ✅ Yes — bypassed by direct SQL |
| ORM validation | `validates :amount, numericality: { greater_than: 0 }` | ✅ Yes — bypassed by direct SQL |

> **Rule**: Any invariant that must *always* hold should be enforced in the **database**, not just in the application.

---

## Best Practices

- Define `CHECK` constraints for every domain rule that can be expressed as a column-level condition.
- Use `NOT NULL` by default — only allow `NULL` when "absence of a value" is genuinely meaningful.
- Use `FOREIGN KEY` constraints for every relationship — let the database maintain referential integrity.
- Encode critical business invariants (e.g., "total money is conserved") as triggers or as assertions in your test suite.
- Test constraint violations explicitly — write tests that try to insert bad data and verify it is rejected.

## Common Mistakes

- Relying solely on application-level validation — a maintenance script, migration, or bug can insert invalid data.
- Using `ENUM` for values that will change — adding a new option requires `ALTER TABLE`.
- Forgetting `ON DELETE RESTRICT` on foreign keys — allows parent rows to be deleted, leaving orphan children.
- Not testing constraint violations — discovering them in production.

## Next Step

Continue to [5-isolation.md](5-isolation.md) for the third ACID property — Isolation.