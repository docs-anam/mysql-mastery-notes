# Index Fundamentals

## Why Indexes Matter

Indexes speed up lookups and sorting by reducing scanned rows.

## Basic Index Example

```sql
CREATE INDEX idx_customers_email ON customers(email);
```

## Composite Index Example

```sql
CREATE INDEX idx_orders_customer_created_at
ON orders(customer_id, created_at);
```

## Caution

More indexes can slow inserts and updates, so build indexes around real query patterns.

## Next Step

Continue to [33-full-text-search.md](33-full-text-search.md).
