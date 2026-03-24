# Entity Relationship Design

## Entities Overview

| Entity | Description |
|--------|-------------|
| `users` | Customers who place orders |
| `products` | The product catalog (source for snapshots) |
| `coupons` | Discount codes that can be applied to orders |
| `orders` | One order per purchase session; contains totals and shipping snapshot |
| `order_items` | Line items within an order; contains product snapshot |
| `order_status_logs` | Audit trail of every status transition for an order |

---

## Entity Details

### `users`

Stores customer accounts. Referenced by `orders.user_id`.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `BIGINT UNSIGNED PK` | |
| `username` | `VARCHAR(50) UNIQUE` | |
| `email` | `VARCHAR(255) UNIQUE` | |
| `full_name` | `VARCHAR(150)` | |
| `phone` | `VARCHAR(20) NULL` | |
| `is_active` | `BOOLEAN` | |
| `created_at` | `DATETIME` | |

---

### `products`

Live product data. `order_items` snapshots from this table at order time.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `BIGINT UNSIGNED PK` | |
| `sku` | `VARCHAR(100) UNIQUE` | |
| `name` | `VARCHAR(255)` | May change — that's why we snapshot |
| `price` | `DECIMAL(15,2)` | May change — that's why we snapshot |
| `stock` | `INT UNSIGNED` | |
| `is_active` | `BOOLEAN` | |
| `created_at` / `updated_at` | `DATETIME` | |

---

### `coupons`

Discount code definitions. Orders reference them by code string (not by FK).

| Column | Type | Notes |
|--------|------|-------|
| `id` | `INT UNSIGNED PK` | |
| `code` | `VARCHAR(50) UNIQUE` | The string stored on orders |
| `discount_type` | `ENUM('fixed', 'percent')` | Fixed amount or percentage off |
| `discount_value` | `DECIMAL(15,2)` | Amount or percentage |
| `min_order_amount` | `DECIMAL(15,2) NULL` | Minimum order total to be eligible |
| `max_uses` | `INT UNSIGNED NULL` | NULL = unlimited |
| `used_count` | `INT UNSIGNED` | Incremented on each use |
| `valid_from` / `valid_until` | `DATETIME NULL` | Validity window |
| `is_active` | `BOOLEAN` | |

---

### `orders`

The central order record. Contains the shipping address and coupon snapshots.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `BIGINT UNSIGNED PK` | |
| `order_number` | `VARCHAR(30) UNIQUE` | Human-readable (e.g., `ORD-20260310-00042`) |
| `user_id` | `BIGINT UNSIGNED FK → users` | |
| `status` | `ENUM(...)` | Current status for fast lookup |
| `subtotal` | `DECIMAL(15,2)` | Sum of all line_totals before discounts |
| `shipping_cost` | `DECIMAL(15,2)` | |
| `discount_amount` | `DECIMAL(15,2)` | Computed from coupon at order time |
| `tax_amount` | `DECIMAL(15,2)` | |
| `grand_total` | `DECIMAL(15,2)` | Final amount charged |
| `coupon_code` | `VARCHAR(50) NULL` | Snapshot — not a FK |
| `payment_method` | `VARCHAR(50) NULL` | e.g., `bank_transfer`, `credit_card` |
| `payment_ref` | `VARCHAR(100) NULL` | Gateway transaction ID |
| **Shipping address (snapshot)** | | |
| `ship_full_name` | `VARCHAR(150)` | Recipient name at time of order |
| `ship_phone` | `VARCHAR(20)` | |
| `ship_address_line1` | `VARCHAR(255)` | |
| `ship_address_line2` | `VARCHAR(255) NULL` | |
| `ship_city` | `VARCHAR(100)` | |
| `ship_province` | `VARCHAR(100)` | |
| `ship_postal_code` | `VARCHAR(20)` | |
| `ship_country` | `VARCHAR(100)` | |
| `notes` | `TEXT NULL` | Customer notes on the order |
| `ordered_at` | `DATETIME` | When the order was placed |
| `updated_at` | `DATETIME` | |

---

### `order_items`

Line items. One row per product per order. Contains product snapshot.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `BIGINT UNSIGNED PK` | |
| `order_id` | `BIGINT UNSIGNED FK → orders` | |
| `product_id` | `BIGINT UNSIGNED` | Soft reference — no FK constraint |
| `product_name` | `VARCHAR(255)` | **Snapshot** |
| `product_sku` | `VARCHAR(100)` | **Snapshot** |
| `unit_price` | `DECIMAL(15,2)` | **Snapshot** — price at order time |
| `quantity` | `INT UNSIGNED` | |
| `line_total` | `DECIMAL(15,2)` | `unit_price * quantity` — pre-computed |

---

### `order_status_logs`

Immutable audit trail. A new row is appended on every status transition.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `BIGINT UNSIGNED PK` | |
| `order_id` | `BIGINT UNSIGNED FK → orders` | |
| `status` | `ENUM(...)` | The status moved TO |
| `note` | `VARCHAR(500) NULL` | Optional reason / courier info |
| `created_by` | `VARCHAR(100) NULL` | Who/what triggered the transition |
| `created_at` | `DATETIME` | When the transition occurred |

---

## Entity Relationship Diagram

```
users
 │  id (PK)
 │  username
 │  email
 │  full_name
 │
 │  1
 │  │
 │  │ places
 │  │
 │  ∞
 ▼
orders
 │  id (PK)
 │  order_number (UNIQUE)
 │  user_id (FK → users)
 │  status
 │  subtotal / shipping_cost / discount_amount / tax_amount / grand_total
 │  coupon_code (snapshot)
 │  ship_* columns (address snapshot)
 │  ordered_at
 │
 ├──────────────────────────── 1:N ──────────────────────►
 │                                                order_status_logs
 │                                                 id (PK)
 │                                                 order_id (FK → orders)
 │                                                 status
 │                                                 note
 │                                                 created_at
 │
 │  1
 │  │
 │  │ contains
 │  │
 │  ∞
 ▼
order_items
   id (PK)
   order_id (FK → orders)
   product_id (soft ref — no FK)
   product_name  ◄── snapshot from products.name
   product_sku   ◄── snapshot from products.sku
   unit_price    ◄── snapshot from products.price
   quantity
   line_total


products (source for snapshots — no hard FK from order_items)
   id
   sku
   name
   price
   ...

coupons (source for discount — no hard FK from orders)
   id
   code  ◄──────────── orders.coupon_code (string snapshot)
   discount_type
   discount_value
   ...
```

---

## Relationship Summary

| From | To | Type | FK? | Notes |
|------|----|------|-----|-------|
| `users` | `orders` | 1:N | Hard FK | One user has many orders |
| `orders` | `order_items` | 1:N | Hard FK | One order has many items |
| `orders` | `order_status_logs` | 1:N | Hard FK | One order has many log entries |
| `order_items` | `products` | N:1 | **No FK** | Soft reference; snapshot used instead |
| `orders` | `coupons` | N:1 | **No FK** | Coupon code stored as string snapshot |
