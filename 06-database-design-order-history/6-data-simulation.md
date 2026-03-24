# Data Simulation

## 1. Seed Base Data

### Users

```sql
INSERT INTO users (username, email, full_name, phone) VALUES
('alice',   'alice@example.com',   'Alice Santoso',  '081200000001'),
('bob',     'bob@example.com',     'Bob Prasetyo',   '081200000002'),
('charlie', 'charlie@example.com', 'Charlie Wijaya', NULL);
```

### Products

```sql
INSERT INTO products (id, sku, name, price, stock) VALUES
(1, 'SHOE-RUN-001',  'AeroStride 200',    899000.00,  150),
(2, 'SHOE-RUN-002',  'AeroStride Pro',   1299000.00,   80),
(3, 'PHONE-X-001',   'Lumina X1',        8999000.00,   40),
(4, 'PHONE-Y-002',   'Lumina X1 Ultra', 12499000.00,   25);
```

### Coupons

```sql
INSERT INTO coupons
    (code, discount_type, discount_value, min_order_amount, max_uses, valid_from, valid_until)
VALUES
('WELCOME10', 'percent', 10.00,  NULL,       NULL, NULL, NULL),
('FLASH20',   'percent', 20.00, 10000000.00, 100, '2026-02-01 00:00:00', '2026-03-31 23:59:59'),
('SAVE50K',   'fixed',   50000.00, 500000.00, NULL, NULL, NULL);
```

---

## 2. Simulate: Placing an Order

The application reads current product data, computes totals, then inserts the order transactionally.

### Order 1 — Alice buys 2x AeroStride 200 with SAVE50K coupon

```sql
START TRANSACTION;

-- Step 1: Create the order shell
INSERT INTO orders (
    order_number, user_id, status,
    subtotal, shipping_cost, discount_amount, tax_amount, grand_total,
    coupon_code,
    payment_method,
    ship_full_name, ship_phone,
    ship_address_line1, ship_city, ship_province, ship_postal_code, ship_country,
    ordered_at
) VALUES (
    'ORD-20260310-00001', 1, 'pending',
    1798000.00, 25000.00, 50000.00, 0.00, 1773000.00,
    'SAVE50K',
    'bank_transfer',
    'Alice Santoso', '081200000001',
    'Jl. Mawar No. 12', 'Jakarta Selatan', 'DKI Jakarta', '12140', 'Indonesia',
    '2026-03-10 09:14:22'
);

SET @order_id = LAST_INSERT_ID();

-- Step 2: Insert the line item (snapshot name, sku, price from products table)
INSERT INTO order_items (order_id, product_id, product_name, product_sku, unit_price, quantity, line_total)
VALUES (@order_id, 1, 'AeroStride 200', 'SHOE-RUN-001', 899000.00, 2, 1798000.00);

-- Step 3: Insert the initial status log entry
INSERT INTO order_status_logs (order_id, status, created_by)
VALUES (@order_id, 'pending', 'system');

-- Step 4: Increment coupon usage
UPDATE coupons SET used_count = used_count + 1 WHERE code = 'SAVE50K';

COMMIT;
```

> **Subtotal check**: 2 × 899,000 = 1,798,000  
> **Grand total**: 1,798,000 + 25,000 − 50,000 + 0 = **1,773,000** ✅

---

### Order 2 — Bob buys Lumina X1 Ultra with FLASH20 coupon

```sql
START TRANSACTION;

INSERT INTO orders (
    order_number, user_id, status,
    subtotal, shipping_cost, discount_amount, tax_amount, grand_total,
    coupon_code,
    payment_method,
    ship_full_name, ship_phone,
    ship_address_line1, ship_city, ship_province, ship_postal_code, ship_country,
    ordered_at
) VALUES (
    'ORD-20260228-00001', 2, 'pending',
    12499000.00, 0.00, 2499800.00, 0.00, 9999200.00,
    'FLASH20',
    'credit_card',
    'Bob Prasetyo', '081200000002',
    'Jl. Melati No. 7', 'Surabaya', 'Jawa Timur', '60111', 'Indonesia',
    '2026-02-28 14:30:00'
);

SET @order_id = LAST_INSERT_ID();

INSERT INTO order_items (order_id, product_id, product_name, product_sku, unit_price, quantity, line_total)
VALUES (@order_id, 4, 'Lumina X1 Ultra', 'PHONE-Y-002', 12499000.00, 1, 12499000.00);

INSERT INTO order_status_logs (order_id, status, created_by)
VALUES (@order_id, 'pending', 'system');

UPDATE coupons SET used_count = used_count + 1 WHERE code = 'FLASH20';

COMMIT;
```

> **Discount**: 20% of 12,499,000 = 2,499,800  
> **Grand total**: 12,499,000 + 0 − 2,499,800 + 0 = **9,999,200** ✅

---

### Order 3 — Charlie buys AeroStride 200 + AeroStride Pro (no coupon)

```sql
START TRANSACTION;

INSERT INTO orders (
    order_number, user_id, status,
    subtotal, shipping_cost, discount_amount, tax_amount, grand_total,
    payment_method,
    ship_full_name, ship_phone,
    ship_address_line1, ship_city, ship_province, ship_postal_code, ship_country,
    ordered_at
) VALUES (
    'ORD-20260315-00001', 3, 'pending',
    2198000.00, 35000.00, 0.00, 0.00, 2233000.00,
    'bank_transfer',
    'Charlie Wijaya', '081200000099',
    'Jl. Anggrek No. 3', 'Bandung', 'Jawa Barat', '40111', 'Indonesia',
    '2026-03-15 11:05:00'
);

SET @order_id = LAST_INSERT_ID();

INSERT INTO order_items (order_id, product_id, product_name, product_sku, unit_price, quantity, line_total)
VALUES
    (@order_id, 1, 'AeroStride 200', 'SHOE-RUN-001',  899000.00, 1,  899000.00),
    (@order_id, 2, 'AeroStride Pro', 'SHOE-RUN-002', 1299000.00, 1, 1299000.00);

INSERT INTO order_status_logs (order_id, status, created_by)
VALUES (@order_id, 'pending', 'system');

COMMIT;
```

> **Subtotal**: 899,000 + 1,299,000 = **2,198,000** ✅

---

## 3. Simulate: Order Status Transitions (Order 1)

```sql
-- Payment confirmed
UPDATE orders SET status = 'paid' WHERE id = 1;
-- Trigger inserts log row automatically

-- Processing started
UPDATE orders SET status = 'processing' WHERE id = 1;

-- Shipped by courier
UPDATE orders
SET    status      = 'shipped',
       payment_ref = 'JNE00192837'
WHERE  id = 1;

-- Delivered
UPDATE orders SET status = 'delivered' WHERE id = 1;
```

The `trg_order_status_log` trigger inserts a log row after each `UPDATE`. If you want to attach a note (e.g., tracking number), insert the log row manually before updating:

```sql
-- Manually insert log with note, then update (trigger will fire but note will be redundant)
INSERT INTO order_status_logs (order_id, status, note, created_by)
VALUES (1, 'shipped', 'Tracking: JNE00192837 via JNE REG', 'courier_api');

UPDATE orders SET status = 'shipped' WHERE id = 1;
```

---

## 4. Simulate: Price and Product Change (Proving Snapshot Correctness)

```sql
-- Rename the product and raise its price
UPDATE products
SET name  = 'AeroStride 200 v2',
    price = 950000.00
WHERE id = 1;
```

Now query Order 1 line items:

```sql
SELECT
    oi.product_id,
    oi.product_name,    -- snapshot
    oi.product_sku,     -- snapshot
    oi.unit_price,      -- snapshot
    oi.quantity,
    oi.line_total
FROM order_items oi
WHERE oi.order_id = 1;
```

**Result:**

```
product_id | product_name   | product_sku  | unit_price | quantity | line_total
-----------|----------------|--------------|------------|----------|-----------
         1 | AeroStride 200 | SHOE-RUN-001 |  899000.00 |        2 | 1798000.00
```

The historical order still shows `AeroStride 200` at `Rp 899,000` — the original values. The product table now has `AeroStride 200 v2` at `Rp 950,000`, but that does not affect the order.

---

## 5. Queries

### Order history list for a user (most recent first)

```sql
SELECT
    o.id,
    o.order_number,
    o.status,
    o.grand_total,
    o.ordered_at,
    COUNT(oi.id) AS item_count
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
WHERE o.user_id = 1
GROUP BY o.id
ORDER BY o.ordered_at DESC
LIMIT 20;
```

### Full order detail with line items

```sql
SELECT
    o.order_number,
    o.status,
    o.subtotal,
    o.shipping_cost,
    o.discount_amount,
    o.grand_total,
    o.coupon_code,
    o.ship_full_name,
    o.ship_address_line1,
    o.ship_city,
    o.ordered_at,
    oi.product_name,
    oi.product_sku,
    oi.unit_price,
    oi.quantity,
    oi.line_total
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
WHERE o.order_number = 'ORD-20260310-00001';
```

### Full status history for an order

```sql
SELECT
    osl.status,
    osl.note,
    osl.created_by,
    osl.created_at
FROM order_status_logs osl
WHERE osl.order_id = 1
ORDER BY osl.created_at ASC;
```

---

## 6. Analytics Queries

### Daily revenue

```sql
SELECT
    DATE(ordered_at)  AS order_date,
    COUNT(*)          AS order_count,
    SUM(grand_total)  AS daily_revenue
FROM orders
WHERE status NOT IN ('cancelled', 'returned')
GROUP BY DATE(ordered_at)
ORDER BY order_date DESC;
```

### Top-selling products by quantity

```sql
SELECT
    oi.product_sku,
    oi.product_name,
    SUM(oi.quantity)    AS total_qty_sold,
    SUM(oi.line_total)  AS total_revenue
FROM order_items oi
JOIN orders o ON o.id = oi.order_id
WHERE o.status NOT IN ('cancelled', 'returned')
GROUP BY oi.product_sku, oi.product_name
ORDER BY total_qty_sold DESC
LIMIT 10;
```

### Average order value

```sql
SELECT
    ROUND(AVG(grand_total), 2)  AS avg_order_value,
    MIN(grand_total)            AS min_order_value,
    MAX(grand_total)            AS max_order_value
FROM orders
WHERE status NOT IN ('cancelled', 'returned');
```

### Coupon usage summary

```sql
SELECT
    o.coupon_code,
    COUNT(*)              AS times_used,
    SUM(o.discount_amount) AS total_discount_given
FROM orders o
WHERE o.coupon_code IS NOT NULL
  AND o.status NOT IN ('cancelled', 'returned')
GROUP BY o.coupon_code
ORDER BY times_used DESC;
```

---

## Summary: Key Patterns in This Module

| Pattern | Technique |
|---------|-----------|
| Immutable price record | Copy `unit_price` from `products` at order time |
| Product rename safety | Copy `product_name` / `product_sku` at order time |
| Shipping address safety | Copy `ship_*` columns at order time; no FK to address table |
| Coupon history | Store `coupon_code` string + `discount_amount` on order; no FK to `coupons` |
| Order status audit trail | `order_status_logs` + `AFTER UPDATE` trigger on `orders` |
| Consistent totals | `grand_total = subtotal + shipping - discount + tax` enforced at application insert |
| Analytics | All queries use `status NOT IN ('cancelled', 'returned')` filter |
