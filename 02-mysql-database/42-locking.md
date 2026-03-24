# Locking

## Purpose

Locks coordinate concurrent access so data stays consistent under multi-user workloads.

## Common Concepts

- Shared locks for reads
- Exclusive locks for writes
- Row-level locking for finer concurrency

## Example

```sql
START TRANSACTION;
SELECT * FROM orders WHERE id = 100 FOR UPDATE;
UPDATE orders SET status = 'paid' WHERE id = 100;
COMMIT;
```

## Next Step

Continue to [43-user-management.md](43-user-management.md).
