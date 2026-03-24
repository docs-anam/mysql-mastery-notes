# Inbox: Notifications and Delivery

## The Two-Table Architecture

The inbox is built from two closely related tables:

```
notifications                    user_notifications
─────────────────────            ──────────────────────────────
id                ◄──────────── notification_id
title                            id
body                             user_id  ──► users.id
action_url                       is_read
data (JSON)                      read_at
category_id ──► categories       deleted_at
created_at                       created_at
```

| Table | Represents | Row count |
|-------|-----------|-----------|
| `notifications` | The event that happened | 1 per event |
| `user_notifications` | The delivery to a specific user | 1 per user per event |

**Why two tables?**

Consider a "system maintenance" broadcast. If we store the notification body in `user_notifications` directly, we duplicate it for every user in the system. With 10 million users, that is 10 million rows of identical text. Instead:
- Store the body **once** in `notifications`
- Store a **lightweight pointer** (+ read status) per user in `user_notifications`

---

## Create the notifications Table

```sql
CREATE TABLE notifications (
    id          BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    title       VARCHAR(255)     NOT NULL,
    body        TEXT             NULL,
    action_url  VARCHAR(512)     NULL,
    data        JSON             NULL,
    category_id INT UNSIGNED     NOT NULL,
    created_at  DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    INDEX idx_notif_category (category_id),
    INDEX idx_notif_created  (created_at),
    CONSTRAINT fk_notif_category
        FOREIGN KEY (category_id) REFERENCES notification_categories (id)
) ENGINE=InnoDB;
```

### Column Guide

| Column | Type | Purpose |
|--------|------|---------|
| `title` | `VARCHAR(255)` | Short headline shown in the notification badge popup |
| `body` | `TEXT NULL` | Optional longer description (can be NULL for compact push alerts) |
| `action_url` | `VARCHAR(512) NULL` | Deep-link the notification opens when tapped |
| `data` | `JSON NULL` | Arbitrary structured payload for the client (e.g., `{"order_id": 4821}`) |
| `category_id` | `INT UNSIGNED` | Links to `notification_categories`; drives routing and preferences |

---

## Create the user_notifications Table

```sql
CREATE TABLE user_notifications (
    id              BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    user_id         BIGINT UNSIGNED  NOT NULL,
    notification_id BIGINT UNSIGNED  NOT NULL,
    read_at         DATETIME         NULL,
    deleted_at      DATETIME         NULL,
    created_at      DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_user_notif  (user_id, notification_id),

    -- Inbox query index: user + unread + newest first
    INDEX idx_un_user_unread_created (user_id, read_at, created_at),

    -- Lookup by notification (e.g., for broadcast management)
    INDEX idx_un_notification (notification_id),

    CONSTRAINT fk_un_user
        FOREIGN KEY (user_id)         REFERENCES users (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_un_notification
        FOREIGN KEY (notification_id) REFERENCES notifications (id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

### Why `read_at` Instead of `is_read`?

| Column design | Pros | Cons |
|---------------|------|------|
| `is_read BOOLEAN` | Simple `WHERE is_read = FALSE` | No record of when it was read; cannot sort "recently read" |
| `read_at DATETIME NULL` | Stores both state and time; `NULL` means unread | Slightly longer column name |

`NULL` means unread; a timestamp means read. This is the standard pattern.

### Why `deleted_at` Instead of Hard Delete?

Hard-deleting a row would:
1. Decrement the counter incorrectly if the notification was unread
2. Break analytics (monthly notification volume reports)
3. Prevent "undo dismiss" functionality

Soft deletes keep the row but filter it out of inbox queries with `WHERE deleted_at IS NULL`.

---

## Seeding Notifications

```sql
-- Assume categories are seeded as in 4-categories.md
-- 1=social, 2=transactional, 3=marketing, 4=system

INSERT INTO notifications (title, body, action_url, data, category_id) VALUES
(
    'Alice liked your post',
    'Alice Santoso liked your post "A day in Bali"',
    '/posts/42',
    '{"liker_id": 1, "post_id": 42}',
    1   -- social
),
(
    'Your order #4821 has shipped',
    'Your order is on the way! Estimated delivery: 2 days.',
    '/orders/4821',
    '{"order_id": 4821, "courier": "JNE", "tracking": "JNE0012345"}',
    2   -- transactional
),
(
    '🎉 Weekend Sale — Up to 50% off!',
    'Shop our biggest weekend sale starting now.',
    '/promo/weekend-sale',
    '{"promo_code": "SALE50"}',
    3   -- marketing
),
(
    'Scheduled maintenance tonight',
    'The platform will be down 02:00 – 04:00 WIB for scheduled maintenance.',
    NULL,
    NULL,
    4   -- system
);

-- Deliver notifications 1 and 2 to Bob (id=2)
-- Deliver notification 3 to Alice (id=1) and Bob (id=2)
-- Deliver notification 4 (system) to everyone

INSERT INTO user_notifications (user_id, notification_id) VALUES
(2, 1),  -- Bob receives "Alice liked your post"
(2, 2),  -- Bob receives "Order shipped"
(1, 3),  -- Alice receives marketing
(2, 3),  -- Bob receives marketing
(1, 4),  -- Alice receives system
(2, 4),  -- Bob receives system
(3, 4),  -- Charlie receives system
(4, 4),  -- Diana receives system
(5, 4);  -- Evan receives system
```

---

## Core Inbox Queries

### Get Bob's full inbox (newest first, exclude deleted)

```sql
SELECT
    un.id           AS inbox_id,
    n.title,
    n.body,
    n.action_url,
    c.name          AS category,
    un.read_at,
    un.created_at
FROM user_notifications un
JOIN notifications n ON n.id = un.notification_id
JOIN notification_categories c ON c.id = n.category_id
WHERE un.user_id    = 2           -- Bob
  AND un.deleted_at IS NULL       -- not deleted
ORDER BY un.created_at DESC
LIMIT 20 OFFSET 0;
```

### Get Bob's unread notifications only

```sql
SELECT
    un.id,
    n.title,
    c.name AS category,
    un.created_at
FROM user_notifications un
JOIN notifications n ON n.id = un.notification_id
JOIN notification_categories c ON c.id = n.category_id
WHERE un.user_id    = 2
  AND un.read_at    IS NULL       -- unread
  AND un.deleted_at IS NULL
ORDER BY un.created_at DESC;
```

### Get Bob's social notifications only

```sql
SELECT
    un.id,
    n.title,
    un.read_at,
    un.created_at
FROM user_notifications un
JOIN notifications n ON n.id = un.notification_id
WHERE un.user_id    = 2
  AND n.category_id = 1           -- social
  AND un.deleted_at IS NULL
ORDER BY un.created_at DESC
LIMIT 20;
```

---

## Index Strategy Explained

```sql
INDEX idx_un_user_unread_created (user_id, read_at, created_at)
```

This composite index covers the most frequent query: *"Give me all unread notifications for user X, newest first."*

1. `user_id` — narrows to one user's rows (high cardinality filter first)
2. `read_at` — filters `IS NULL` (unread) using the index
3. `created_at` — used for `ORDER BY` without a file sort

Without this index, MySQL would full-scan `user_notifications` for every inbox page load.

---

## Best Practices

- Always `JOIN notifications` instead of copying notification content into `user_notifications` — one source of truth.
- Always filter `deleted_at IS NULL` in inbox queries; never assume the application already filters this.
- Use cursor-based pagination (`WHERE created_at < :last_seen_created_at`) for large inboxes instead of `OFFSET`, which degrades as offset grows.

## Common Mistakes

- Storing the notification title/body in `user_notifications` — creates millions of duplicate rows for broadcast notifications.
- Forgetting the `UNIQUE KEY (user_id, notification_id)` — allows a user to receive the same notification twice.
- Hard-deleting `user_notifications` rows — breaks counter accuracy and prevents undo-dismiss.

## Next Step

Continue to [7-read-status.md](7-read-status.md) to implement read/unread status tracking.