# Implementation

## Create the Database

```sql
CREATE DATABASE IF NOT EXISTS order_history
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE order_history;
```

---

## 1. `users`

```sql
CREATE TABLE users (
    id          BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    username    VARCHAR(50)      NOT NULL,
    email       VARCHAR(255)     NOT NULL,
    full_name   VARCHAR(150)     NOT NULL,
    phone       VARCHAR(20)      NULL,
    is_active   BOOLEAN          NOT NULL DEFAULT TRUE,
    created_at  DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP
                                      ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_users_username (username),
    UNIQUE KEY uq_users_email    (email)
) ENGINE=InnoDB;
```

---

## 2. `products`

```sql
CREATE TABLE products (
    id          BIGINT UNSIGNED   NOT NULL AUTO_INCREMENT,
    sku         VARCHAR(100)      NOT NULL,
    name        VARCHAR(255)      NOT NULL,
    description TEXT              NULL,
    price       DECIMAL(15, 2)    NOT NULL,
    stock       INT UNSIGNED      NOT NULL DEFAULT 0,
    is_active   BOOLEAN           NOT NULL DEFAULT TRUE,
    created_at  DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP
                                       ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_products_sku (sku),
    INDEX idx_products_active  (is_active)
) ENGINE=InnoDB;
```

---

## 3. `coupons`

```sql
CREATE TABLE coupons (
    id               INT UNSIGNED      NOT NULL AUTO_INCREMENT,
    code             VARCHAR(50)       NOT NULL,
    discount_type    ENUM('fixed','percent') NOT NULL,
    discount_value   DECIMAL(15, 2)    NOT NULL,
    min_order_amount DECIMAL(15, 2)    NULL,
    max_uses         INT UNSIGNED      NULL,
    used_count       INT UNSIGNED      NOT NULL DEFAULT 0,
    valid_from       DATETIME          NULL,
    valid_until      DATETIME          NULL,
    is_active        BOOLEAN           NOT NULL DEFAULT TRUE,
    created_at       DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_coupons_code (code),
    INDEX idx_coupons_active   (is_active),

    CONSTRAINT chk_discount_value   CHECK (discount_value > 0),
    CONSTRAINT chk_discount_percent CHECK (
        discount_type <> 'percent' OR discount_value <= 100
    )
) ENGINE=InnoDB;
```

### Design Notes

| Column | Reason |
|--------|--------|
| `discount_type ENUM` | Avoids magic numbers; enforces only `fixed` or `percent` at the DB level |
| `CHECK (discount_value > 0)` | A zero or negative discount makes no semantic sense |
| `CHECK … discount_value <= 100` | A percentage coupon cannot exceed 100% |
| `used_count` | Updated atomically on each order; used with `max_uses` to throttle usage |
| `valid_from / valid_until NULL` | NULL means no date restriction in that direction |

---

## 4. `orders`

```sql
CREATE TABLE orders (
    id               BIGINT UNSIGNED   NOT NULL AUTO_INCREMENT,
    order_number     VARCHAR(30)       NOT NULL,
    user_id          BIGINT UNSIGNED   NOT NULL,
    status           ENUM(
                         'pending',
                         'paid',
                         'processing',
                         'shipped',
                         'delivered',
                         'cancelled',
                         'returned'
                     )                 NOT NULL DEFAULT 'pending',

    -- Totals
    subtotal         DECIMAL(15, 2)    NOT NULL,
    shipping_cost    DECIMAL(15, 2)    NOT NULL DEFAULT 0.00,
    discount_amount  DECIMAL(15, 2)    NOT NULL DEFAULT 0.00,
    tax_amount       DECIMAL(15, 2)    NOT NULL DEFAULT 0.00,
    grand_total      DECIMAL(15, 2)    NOT NULL,

    -- Coupon snapshot (no FK — coupon may be deleted later)
    coupon_code      VARCHAR(50)       NULL,

    -- Payment
    payment_method   VARCHAR(50)       NULL,
    payment_ref      VARCHAR(100)      NULL,

    -- Shipping address snapshot
    ship_full_name      VARCHAR(150)   NOT NULL,
    ship_phone          VARCHAR(20)    NOT NULL,
    ship_address_line1  VARCHAR(255)   NOT NULL,
    ship_address_line2  VARCHAR(255)   NULL,
    ship_city           VARCHAR(100)   NOT NULL,
    ship_province       VARCHAR(100)   NOT NULL,
    ship_postal_code    VARCHAR(20)    NOT NULL,
    ship_country        VARCHAR(100)   NOT NULL DEFAULT 'Indonesia',

    notes            TEXT              NULL,
    ordered_at       DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at       DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP
                                            ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_orders_number     (order_number),
    INDEX idx_orders_user           (user_id),
    INDEX idx_orders_status         (status),
    INDEX idx_orders_ordered_at     (ordered_at),

    -- Composite index for order history list query (user + newest first)
    INDEX idx_orders_user_date      (user_id, ordered_at DESC),

    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id) REFERENCES users (id),

    CONSTRAINT chk_orders_grand_total  CHECK (grand_total >= 0),
    CONSTRAINT chk_orders_subtotal     CHECK (subtotal    >= 0),
    CONSTRAINT chk_orders_discount     CHECK (discount_amount >= 0)
) ENGINE=InnoDB;
```

### Grand Total Formula

```
grand_total = subtotal + shipping_cost - discount_amount + tax_amount
```

This is enforced by the application at insert time, not by a `CHECK` constraint (MySQL `CHECK` constraints cannot reference other row values in aggregate expressions).

### Why `order_number` and not just `id`?

| Identifier | Type | Purpose |
|-----------|------|---------|
| `id` | `BIGINT` auto-increment | Internal primary key; used for JOINs |
| `order_number` | `VARCHAR` | Human-readable; shown on receipts, emails, support tickets |

`order_number` format example: `ORD-20260310-00042`  
Generated by the application before the `INSERT` using the current date + a zero-padded sequence.

---

## 5. `order_items`

```sql
CREATE TABLE order_items (
    id            BIGINT UNSIGNED   NOT NULL AUTO_INCREMENT,
    order_id      BIGINT UNSIGNED   NOT NULL,

    -- Soft reference to the product (informational, no FK)
    product_id    BIGINT UNSIGNED   NOT NULL,

    -- Snapshots captured at order placement time
    product_name  VARCHAR(255)      NOT NULL,
    product_sku   VARCHAR(100)      NOT NULL,
    unit_price    DECIMAL(15, 2)    NOT NULL,

    quantity      INT UNSIGNED      NOT NULL,
    line_total    DECIMAL(15, 2)    NOT NULL,  -- unit_price * quantity

    PRIMARY KEY (id),
    INDEX idx_oi_order   (order_id),
    INDEX idx_oi_product (product_id),

    CONSTRAINT fk_oi_order
        FOREIGN KEY (order_id) REFERENCES orders (id)
        ON DELETE CASCADE,

    CONSTRAINT chk_oi_quantity   CHECK (quantity   > 0),
    CONSTRAINT chk_oi_unit_price CHECK (unit_price > 0),
    CONSTRAINT chk_oi_line_total CHECK (line_total > 0)
) ENGINE=InnoDB;
```

### Why `ON DELETE CASCADE` on `order_id`?

If an order is hard-deleted (e.g., during a test environment purge), its line items should be removed too. In production, orders are typically **never deleted** — they are set to `cancelled` or `returned`. The cascade is a safety net.

### Why no FK on `product_id`?

See `3-history-vs-non-history.md`. Briefly: a hard FK would prevent admins from deleting discontinued products from the catalog.

---

## 6. `order_status_logs`

```sql
CREATE TABLE order_status_logs (
    id          BIGINT UNSIGNED   NOT NULL AUTO_INCREMENT,
    order_id    BIGINT UNSIGNED   NOT NULL,
    status      ENUM(
                    'pending',
                    'paid',
                    'processing',
                    'shipped',
                    'delivered',
                    'cancelled',
                    'returned'
                )                 NOT NULL,
    note        VARCHAR(500)      NULL,
    created_by  VARCHAR(100)      NULL,   -- e.g. 'system', 'admin:5', 'courier_api'
    created_at  DATETIME          NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    INDEX idx_osl_order      (order_id),
    INDEX idx_osl_order_time (order_id, created_at),

    CONSTRAINT fk_osl_order
        FOREIGN KEY (order_id) REFERENCES orders (id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

### Why a Separate Log Table?

| Approach | Pros | Cons |
|----------|------|------|
| Single `status` column on `orders` | Simple | No history; cannot reconstruct "how long did it stay in `processing`?" |
| Separate `order_status_logs` | Full audit trail; supports SLA reporting | One extra table and insert per transition |

The log table is a **write-once, never-update** audit log. No rows are ever updated or deleted from it.

---

## Trigger: Auto-Log on Status Change

When `orders.status` is updated, automatically append a row to `order_status_logs`.

```sql
DELIMITER $$

CREATE TRIGGER trg_order_status_log
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
    IF NEW.status <> OLD.status THEN
        INSERT INTO order_status_logs (order_id, status, created_by)
        VALUES (NEW.id, NEW.status, 'system');
    END IF;
END $$

DELIMITER ;
```

> Applications that want to add a `note` or `created_by` to the log entry should insert the log row manually before updating `orders.status`, then disable/skip the trigger for that session using a session variable flag if needed. For simplicity in this module, the trigger handles all transitions.

---

## Full Table Creation Order

Because of foreign key dependencies, tables must be created in this order:

1. `users`
2. `products`
3. `coupons`
4. `orders` (FK → `users`)
5. `order_items` (FK → `orders`)
6. `order_status_logs` (FK → `orders`)
