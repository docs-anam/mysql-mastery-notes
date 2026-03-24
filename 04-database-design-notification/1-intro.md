# Database Design: Notification System

## What We Are Building

A **notification system** is one of the most common — and most deceptively difficult — database design problems in modern applications. Every social media platform, e-commerce site, and SaaS product has one.

You have seen it in action:

```
🔔  You have 4 new notifications

  ✉️  Alice liked your post              2 min ago   [unread]
  📦  Your order #4821 has shipped       1 hr ago    [unread]
  👤  Bob started following you          3 hrs ago   [unread]
  🎉  You earned the "Explorer" badge    Yesterday   [read]
```

### Why This Is More Complex Than It Looks

| Challenge | Description |
|-----------|-------------|
| **Fanout** | One event (e.g., a celebrity posts) may trigger notifications for millions of users |
| **Unread counter** | Badge count must be fast — it is queried on every page load |
| **Read tracking** | Marking one item read must not affect others |
| **Category filtering** | Users can opt out of marketing but still receive transactional alerts |
| **Ordering** | Most recent first, always; no pagination gaps |
| **Soft delete** | Notifications are usually hidden, not deleted |

---

## What You Will Build in This Module

A fully functional notification system database for a social + e-commerce application:

```
┌────────────────────────────────────────────────────────────┐
│                    Notification System                     │
│                                                            │
│  users ──────────────────────────────────────┐            │
│     │                                        │            │
│     ├─── user_notification_preferences       │            │
│     │         (per category opt-in/out)      │            │
│     │                                        ▼            │
│     └──────────────────── user_notifications (inbox)      │
│                                   │                        │
│  notification_categories ─────────┼── notifications        │
│     (social, system, marketing)   │    (the event record)  │
│                                   │                        │
│  notification_counters ───────────┘                        │
│     (cached unread badge count)                            │
└────────────────────────────────────────────────────────────┘
```

---

## Learning Path

| Step | File | Topic |
|------|------|-------|
| 1 | `1-intro.md` | Overview, challenges, schema map |
| 2 | `2-requirements.md` | Functional and non-functional requirements |
| 3 | `3-setup.md` | Database setup and users table |
| 4 | `4-categories.md` | Notification categories and routing |
| 5 | `5-preferences.md` | User notification preferences |
| 6 | `6-inbox.md` | Notifications and user_notifications (inbox) |
| 7 | `7-read-status.md` | Read/unread status tracking |
| 8 | `8-counter.md` | Unread counter optimisation |

---

## Key Design Principles

### Separation of Event and Delivery

A single notification **event** (e.g., "Alice liked post #99") can be delivered to multiple users. We store the event once and create a per-user delivery record:

```
notifications (1 row)     →    user_notifications (N rows)
"Alice liked post #99"         user 7: unread
                               user 12: read
                               user 45: unread
```

This is the key architectural decision that avoids data duplication.

### Denormalised Counter

Counting `SELECT COUNT(*) WHERE is_read = FALSE` across millions of rows on every page load is too slow. We maintain a separate counter that is incremented and decremented in real time.

### Soft Deletions

Notifications are never hard-deleted. Instead they have a `deleted_at` timestamp so:
- Users can restore accidentally dismissed notifications
- Analytics can still count historical notification volume
- Audit trails remain intact

---

## Prerequisites

- Solid understanding of MySQL fundamentals (`02-mysql-database/`)
- ACID properties and transactions (`03-mysql-database-acid/`)
- Comfort with `JOIN`, `GROUP BY`, and subqueries

## Next Step

Continue to [2-requirements.md](2-requirements.md) to define exactly what the system must do.