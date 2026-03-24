# Transactions

## Why Transactions

Transactions make a group of SQL statements succeed or fail as one unit.

## Basic Flow

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 50000 WHERE id = 1;
UPDATE accounts SET balance = balance + 50000 WHERE id = 2;
COMMIT;
```

## Rollback Example

```sql
START TRANSACTION;
UPDATE inventory SET stock = stock - 1 WHERE product_id = 10;
-- if error happens
ROLLBACK;
```

## Next Step

Continue to [42-locking.md](42-locking.md).
