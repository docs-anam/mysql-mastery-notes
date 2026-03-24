# Requirement Specification

---

## Actors

```
┌──────────────┐   places order / views history   ┌──────────────────┐
│  Customer    │ ──────────────────────────────►  │  Application     │
│              │ ◄──────────────────────────────  │  (backend API)   │
└──────────────┘   order confirmation / history   └──────┬───────────┘
                                                         │ reads / writes
                                                         ▼
┌──────────────┐   updates order status           ┌──────────────────┐
│  Operations  │ ──────────────────────────────►  │  Database        │
│  Team /      │                                  │  (this module)   │
│  Courier API │                                  └──────────────────┘
└──────────────┘
```

| Actor | Role |
|-------|------|
| **Customer** | Places orders, views order history, tracks shipment status |
| **Application** | Creates order records, computes totals, applies coupons |
| **Operations / Courier API** | Updates order status through the lifecycle |
| **Database** | Persists the permanent, immutable order record and its history |

---

## Functional Requirements

### FR-1: Order Placement

- An order must record: customer, shipping address, order date, and overall status.
- An order may contain one or more **line items** (products purchased).
- Each line item must store: a reference to the product, the **snapshot name**, **snapshot SKU**, **unit price at time of purchase**, and quantity.
- The order must record: subtotal, shipping cost, discount amount, tax amount, and grand total.
- All monetary values must use exact decimal arithmetic (no floating-point).

### FR-2: Price and Product Snapshot

- The unit price stored on each line item must **never change** after the order is created, regardless of future product price changes.
- The product name and SKU stored on each line item must reflect the values **at the time of purchase**.
- Future updates to the `products` table must not alter any existing order_item record.

### FR-3: Order Status Lifecycle

Orders must move through the following states:

```
pending ──► paid ──► processing ──► shipped ──► delivered
              │                         │
              └──► cancelled            └──► returned
```

- Each status transition must be recorded with a timestamp and an optional note.
- The current status is stored on the `orders` table for fast lookup.
- The **full state history** is stored in a separate `order_status_logs` table.

### FR-4: Coupon / Discount

- A coupon code may be applied to an order at placement time.
- The coupon code string and the computed discount amount must both be stored on the order.
- Even if the coupon is later deleted from the system, the order record must retain the code and discount amount.

### FR-5: Shipping Address Snapshot

- The shipping address at the time of ordering must be stored on the order.
- Future changes to the customer's address book must not alter existing order records.
- The address is stored as a snapshot (individual columns) on the `orders` table, not as a foreign key to an `addresses` table.

### FR-6: Order History Queries

- A customer must be able to retrieve all their orders, ordered by most recent first.
- Each order in the list must show: order number, date, grand total, and current status.
- A customer must be able to view the full detail of a single order, including all line items.

### FR-7: Order Analytics

- It must be possible to query: total revenue per day/month, top-selling products by quantity and revenue, average order value.
- These queries must be performable without custom reporting tables (using indexes on the main schema).

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| **Order placement write** | Full order insert (order + items + log entry) < 50ms |
| **Order history list** | Paginated 20-row result < 20ms |
| **Order detail lookup** | Single order + all items < 10ms |
| **Financial accuracy** | All money stored as `DECIMAL` — no floating-point |
| **Immutability** | unit_price and snapshot fields on order_items never updated after insert |
| **Auditability** | Every status change recorded with timestamp in `order_status_logs` |

---

## Out of Scope

- Payment gateway integration (payment method, gateway transaction ID are noted but not the gateway flow)
- Product inventory decrement / reservation (separate concern)
- Return / refund workflows beyond status recording
- Multi-currency pricing
- Seller / vendor split in a marketplace model
