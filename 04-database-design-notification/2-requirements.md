# Requirement Specification

Before touching a single `CREATE TABLE`, we must understand exactly what we are building.

---

## Actors

```
┌──────────────┐     trigger event      ┌──────────────────┐
│  Application │ ──────────────────────►│ Notification     │
│  (backend)   │                        │ Service          │
└──────────────┘                        └──────┬───────────┘
                                               │ delivers to
                                               ▼
┌──────────────┐     views / marks read  ┌──────────────────┐
│  User        │ ◄──────────────────────►│ Database         │
│  (browser /  │                        │ (this module)    │
│   mobile)    │                        └──────────────────┘
└──────────────┘
```

| Actor | Role |
|-------|------|
| **Application** | Inserts notification events when domain actions occur |
| **User** | Reads their inbox, marks notifications read/unread/deleted |
| **Database** | Stores, queries, and maintains counter consistency |

---

## Functional Requirements

### FR-1: Notification Delivery

- The system must deliver notifications to one or more target users per event.
- A notification must store: title, body/message, an optional deep-link URL, and an optional JSON payload for the client.
- Each notification belongs to exactly one category.

### FR-2: Inbox Queries

- A user must be able to retrieve their notification inbox, ordered by most recent first.
- The inbox must support **pagination** (limit + offset / cursor).
- The inbox must support filtering by: all / unread only / by category.

### FR-3: Read / Unread Status

- Each notification in a user's inbox has an independent read status.
- A user can mark a single notification as read.
- A user can mark **all** notifications as read in one operation.
- The system must record **when** a notification was read (timestamp, not just boolean).

### FR-4: Unread Counter

- Each user has an unread notification count (the "badge number").
- The count must be retrievable in a single fast query (no full table scan).
- The count must automatically increment when a new notification is delivered.
- The count must automatically decrement when a notification is marked as read.
- The count must reset to zero when "mark all as read" is executed.

### FR-5: Categories

- Notifications must belong to a category (e.g., `social`, `transactional`, `marketing`, `system`).
- Users must be able to **opt out** of specific categories (except `system` — always delivered).
- Opted-out users must not receive new notifications from that category.

### FR-6: Soft Delete

- Users can dismiss/delete a notification without permanently removing it.
- Deleted notifications are hidden from the inbox but remain in the database.
- Deleted notifications are excluded from the unread counter.

---

## Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| **Inbox load time** | < 100ms for the first page (20 items) |
| **Unread count query** | < 5ms |
| **Write throughput** | Able to insert 1,000 notifications per second |
| **History retention** | Notifications retained for 1 year |
| **Concurrency** | Multiple sessions marking read simultaneously must not corrupt the counter |

---

## Use Cases

| ID | Use Case | Actor | Description |
|----|----------|-------|-------------|
| UC-01 | Send notification | Application | Insert event + per-user delivery records |
| UC-02 | View inbox | User | Paginated list of notifications, newest first |
| UC-03 | View unread inbox | User | Filtered to `is_read = FALSE` |
| UC-04 | Mark one read | User | Set `read_at = NOW()` for one `user_notification` |
| UC-05 | Mark all read | User | Set `read_at = NOW()` for all unread + reset counter |
| UC-06 | Delete notification | User | Set `deleted_at = NOW()` |
| UC-07 | Get badge count | User | Return `unread_count` from counter table |
| UC-08 | Opt out of category | User | Insert preference row `is_enabled = FALSE` |
| UC-09 | View by category | User | Filter inbox by `category_id` |

---

## Data Constraints and Business Rules

| Rule | Detail |
|------|--------|
| A notification must have a non-empty title | `NOT NULL`, length 1–255 |
| A notification body can be NULL | Optional extended description |
| `action_url` must be a valid URL or NULL | Application-level validation |
| `amount` in JSON payload is always positive | Application-level validation |
| A user cannot opt out of the `system` category | Enforced at application layer (or DB trigger) |
| Counter never goes below 0 | `CHECK (unread_count >= 0)` |
| `read_at` cannot be earlier than `created_at` | Enforced at application layer |

---

## Out of Scope for This Module

- Push notification delivery (FCM, APNs) — this module is database only
- Email / SMS sending
- Real-time WebSocket delivery
- Notification templates / i18n
- Admin interface

---

## Next Step

Continue to [3-setup.md](3-setup.md) to create the database and foundation tables.