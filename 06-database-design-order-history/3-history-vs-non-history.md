# History Design vs Non-History Design

This is the most important design decision in this module. It determines whether your order records remain correct forever — or become silently wrong as your data evolves.

---

## The Non-History (Live-Join) Approach

Store only the foreign key on `order_items` and JOIN to `products` at query time.

```sql
-- Schema
CREATE TABLE order_items (
    id          BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    order_id    BIGINT UNSIGNED NOT NULL,
    product_id  BIGINT UNSIGNED NOT NULL,   -- FK only
    quantity    INT UNSIGNED    NOT NULL,
    PRIMARY KEY (id)
);

-- Query
SELECT
    p.name,
    p.price,         -- live price from products table
    oi.quantity,
    p.price * oi.quantity AS line_total
FROM order_items oi
JOIN products p ON p.id = oi.product_id;
```

### What goes wrong

| Timeline | Event | Query result |
|----------|-------|-------------|
| Day 1 | Customer buys product, price = Rp 899,000 | Rp 899,000 ✅ |
| Day 30 | Admin updates price to Rp 950,000 | Rp 950,000 ❌ |
| Day 60 | Product renamed "AeroStride 200 v2" | Shows new name ❌ |
| Day 90 | Product deleted (discontinued) | ORDER ITEM QUERY RETURNS NULL ❌ |

Every historical order now shows incorrect data. Financial records are corrupted. Customer invoices are wrong.

### When the live-join approach IS acceptable

| Use case | Acceptable? | Reason |
|----------|------------|--------|
| Shopping cart (not yet ordered) | ✅ Yes | Not a financial record; cart should reflect current prices |
| Wishlist / saved items | ✅ Yes | User expects to see current price |
| Inventory display | ✅ Yes | Current availability matters |
| **Order history / invoices** | ❌ No | Must reflect what was actually charged |

---

## The History (Snapshot) Approach

At the moment the order is placed, **copy** the relevant values from `products` onto the `order_items` row.

```sql
-- Schema
CREATE TABLE order_items (
    id              BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    order_id        BIGINT UNSIGNED  NOT NULL,
    product_id      BIGINT UNSIGNED  NOT NULL,   -- soft reference (informational)
    -- Snapshots captured at order time:
    product_name    VARCHAR(255)     NOT NULL,
    product_sku     VARCHAR(100)     NOT NULL,
    unit_price      DECIMAL(15, 2)   NOT NULL,
    quantity        INT UNSIGNED     NOT NULL,
    line_total      DECIMAL(15, 2)   NOT NULL,
    PRIMARY KEY (id)
);
```

### The insert process (application side)

```sql
-- Step 1: Read current product data
SELECT name, sku, price FROM products WHERE id = :product_id;

-- Step 2: Insert the snapshot into order_items
INSERT INTO order_items
    (order_id, product_id, product_name, product_sku, unit_price, quantity, line_total)
VALUES
    (:order_id, :product_id, :name, :sku, :price, :qty, :price * :qty);
```

After this insert, `unit_price`, `product_name`, and `product_sku` on this row **never change** — even if `products` is updated, renamed, or deleted.

### Permanent correctness

| Timeline | Event | Query result |
|----------|-------|-------------|
| Day 1 | Order placed, snapshot: "AeroStride 200", Rp 899,000 | Rp 899,000 ✅ |
| Day 30 | Admin updates price to Rp 950,000 | Still Rp 899,000 ✅ |
| Day 60 | Product renamed | Still "AeroStride 200" ✅ |
| Day 90 | Product deleted | Order item still readable ✅ |

---

## Side-by-Side Comparison

| Dimension | Live-Join | Snapshot |
|-----------|-----------|----------|
| Schema complexity | Low | Medium |
| Query complexity | Low (simple JOIN) | Low (no JOIN needed for price/name) |
| Historical accuracy | ❌ Breaks on any product update | ✅ Always correct |
| Disk usage | Low | Slightly higher (duplicated text) |
| Suitable for invoices | ❌ No | ✅ Yes |
| Suitable for shopping cart | ✅ Yes | Not needed |
| Product deletion safe | ❌ No (NULL joins) | ✅ Yes |

---

## What About `product_id`?

Even in the snapshot design, we still store `product_id` as a **soft reference**:

```sql
product_id BIGINT UNSIGNED NOT NULL
-- No FOREIGN KEY constraint here — intentional
```

**Why no FK constraint?**

A hard `FOREIGN KEY … ON DELETE RESTRICT` would prevent deleting a product that has ever been ordered. That is usually undesirable — admins should be able to remove discontinued products from the catalog.

A hard `FOREIGN KEY … ON DELETE CASCADE` would delete order items when a product is removed — a catastrophic data loss.

Instead, `product_id` is stored as a **nullable informational reference**. It can be used to:
- Link back to the current product page ("View current version of this product")
- Run analytics joining current product data

But it is never used to derive `name` or `price` for invoice display.

---

## The Two Tables That Always Need Snapshots

For order history systems, apply the snapshot pattern to:

| Entity | Snapshot stored on | Fields to snapshot |
|--------|-------------------|--------------------|
| Product | `order_items` | `product_name`, `product_sku`, `unit_price` |
| Shipping address | `orders` | `ship_name`, `ship_address`, `ship_city`, etc. |
| Coupon | `orders` | `coupon_code`, `discount_amount` |

The shipping address snapshot is equally important — if a customer updates their default address, old orders must still show where the package was actually sent.
