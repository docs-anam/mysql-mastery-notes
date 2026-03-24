# Database Design: Order History System

## What We Are Building

An **order history system** records every purchase a customer makes and preserves a complete, accurate snapshot of what was ordered, at what price, from which seller — even as products change names, prices are updated, or items are discontinued long after the purchase.

You have seen it in action:

```
📦  My Orders

  Order #10042   •  March 10, 2026   •  Rp 1,798,000   •  DELIVERED
  ──────────────────────────────────────────────────────────────────
  2x  AeroStride 200 (Size 42, Black)       @  Rp  899,000   = Rp 1,798,000
  Shipped via JNE REG  •  Tracking: JNE00192837

  Order #9871    •  February 28, 2026  •  Rp 12,499,000  •  COMPLETED
  ──────────────────────────────────────────────────────────────────
  1x  Lumina X1 Ultra (256 GB, Midnight)    @ Rp 12,499,000  = Rp 12,499,000
  Coupon: FLASH20 applied  •  Discount: Rp 2,499,800
```

The challenge: by the time a customer views Order #9871 six months later, the product price may have changed, the product name may have been updated, and the coupon may no longer exist. The order record must still be **100% accurate**.

---

## Why This Is More Complex Than It Looks

| Challenge | Description |
|-----------|-------------|
| **Price immutability** | The price at order time must be preserved, independent of future price changes |
| **Product snapshot** | The product name, SKU, and attributes must be stored as they were at purchase |
| **Discount / coupon history** | Applied coupons and computed discounts must be recorded on the order |
| **Status lifecycle** | An order transitions through states (`pending → paid → shipped → delivered`) |
| **Multi-item orders** | One order can contain many products, each with its own quantity and price |
| **Totals and taxes** | Subtotal, shipping cost, discount, and tax must all be individually recorded |
| **Audit trail** | Every status change should be timestamped and traceable |

---

## The Core Problem: Data Changes Over Time

Consider this scenario:

```
March 10, 2026: Customer buys "AeroStride 200" for Rp 899,000
April  1, 2026: Price is updated to Rp 950,000
April  5, 2026: Product is renamed to "AeroStride 200 v2"
June   1, 2026: Customer views their March order history
```

If the order record only stores `product_id = 1` and we JOIN to the `products` table at query time:

```sql
-- WRONG approach
SELECT p.name, p.price, oi.quantity
FROM order_items oi
JOIN products p ON p.id = oi.product_id;
-- Returns: "AeroStride 200 v2" @ Rp 950,000  ← INCORRECT
```

The invoice would show the **wrong name and wrong price**. This is a fundamental correctness violation in financial records.

The solution is to **snapshot** the price and relevant product data directly onto the order row at the moment of purchase.

---

## What You Will Build in This Module

A complete order history system for an e-commerce platform:

```
┌──────────────────────────────────────────────────────────────────┐
│                     Order History System                         │
│                                                                  │
│  users ──────────────────────────────────────────┐              │
│     │                                            │              │
│     └──────────── orders ◄─── order_status_logs  │              │
│                      │         (state machine)   │              │
│                      │                           │              │
│           order_items │                          │              │
│    (snapshot: name,   │◄──── products            │              │
│      price at time)   │                          │              │
│                       │                          │              │
│           order_coupons│◄──── coupons             │              │
│    (applied discount)  │      (ref only)         │              │
│                                                  │              │
│  addresses ──────────────────────────────────────┘              │
│     (shipping snapshot on order)                                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## Learning Path

| Step | File | Topic |
|------|------|-------|
| 1 | `1-intro.md` | Overview, the snapshot problem, schema map |
| 2 | `2-requirements.md` | Functional & non-functional requirements |
| 3 | `3-history-vs-non-history.md` | Snapshot design vs live-join design — tradeoffs |
| 4 | `4-entity-relationship-design.md` | ERD — all entities and their relationships |
| 5 | `5-implementation.md` | Full DDL — all tables with constraints and indexes |
| 6 | `6-data-simulation.md` | Seed data, order placement, status transitions, queries |
