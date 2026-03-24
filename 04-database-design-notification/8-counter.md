# Unread Counter

## The Problem

Every page in our app shows a notification badge:

```
🔔 4
```

This number must be:
- **Fast to read** — queried on every page load for every logged-in user
- **Always accurate** — must reflect inserts and mark-as-read in real time
- **Concurrent-safe** — two sessions reading/marking simultaneously must not corrupt it

---

## Why a Naive COUNT Is Not Enough

The obvious approach:

```sql
-- "Simple" unread count
SELECT COUNT(*)
FROM user_notifications
WHERE user_id    = :user_id
  AND read_at    IS NULL
  AND deleted_at IS NULL;
```

This works fine with 100 users. At scale:

| Users | Notifications per user | Total rows | COUNT(*) time |
|-------|----------------------|-----------|--------------|
| 1,000 | 500 | 500K | ~2ms |
| 100,000 | 2,000 | 200M | ~800ms |
| 10,000,000 | 2,000 | 20B | too slow |

Even with the index on `(user_id, read_at, created_at)`, scanning all unread rows for a user becomes slow when a user has thousands of unread notifications. And this query runs on **every page load**.

The solution: maintain a **pre-computed counter** that is updated incrementally.

---

## Create the Counter Table

```sql
CREATE TABLE notification_counters (
    user_id         BIGINT UNSIGNED  NOT NULL,
    unread_count    INT UNSIGNED     NOT NULL DEFAULT 0,
    last_updated_at DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP
                                          ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (user_id),
    CONSTRAINT fk_nc_user
        FOREIGN KEY (user_id) REFERENCES users (id)
        ON DELETE CASCADE,
    CONSTRAINT chk_nc_positive CHECK (unread_count >= 0)
) ENGINE=InnoDB;
```

### Design Notes

| Decision | Reason |
|----------|--------|
| `PRIMARY KEY (user_id)` | One row per user; PK lookup is the fastest possible read |
| `INT UNSIGNED` | Cannot go below 0; max ~4 billion (sufficient) |
| `CHECK (unread_count >= 0)` | Database-level guard against bugs causing negative counters |
| `FOREIGN KEY ... ON DELETE CASCADE` | Deleting a user cleans up their counter row automatically |

---

## Initialise Counters for Existing Users

```sql
-- Create a counter row for every existing user
-- Compute the real count from user_notifications as the starting value
INSERT INTO notification_counters (user_id, unread_count)
SELECT
    u.id,
    COALESCE(COUNT(un.id), 0)
FROM users u
LEFT JOIN user_notifications un
    ON un.user_id    = u.id
   AND un.read_at    IS NULL
   AND un.deleted_at IS NULL
GROUP BY u.id
ON DUPLICATE KEY UPDATE unread_count = VALUES(unread_count);
```

---

## Automating Counter Updates with Triggers

### Trigger 1: Increment on new delivery

When a new `user_notifications` row is inserted (a notification is delivered), increment the recipient's counter.

```sql
DELIMITER $$
CREATE TRIGGER trg_increment_counter_after_delivery
AFTER INSERT ON user_notifications
FOR EACH ROW
BEGIN
    INSERT INTO notification_counters (user_id, unread_count)
    VALUES (NEW.user_id, 1)
    ON DUPLICATE KEY UPDATE
        unread_count    = unread_count + 1,
        last_updated_at = NOW();
END$$
DELIMITER ;
```

### Trigger 2: Decrement when notification is read

When `read_at` is set from `NULL` to a timestamp (the notification is marked as read), decrement the counter.

```sql
DELIMITER $$
CREATE TRIGGER trg_decrement_counter_on_read
AFTER UPDATE ON user_notifications
FOR EACH ROW
BEGIN
    -- Notification was just marked as read (read_at changed from NULL to a value)
    IF OLD.read_at IS NULL AND NEW.read_at IS NOT NULL THEN
        UPDATE notification_counters
        SET unread_count    = GREATEST(unread_count - 1, 0),
            last_updated_at = NOW()
        WHERE user_id = NEW.user_id;
    END IF;

    -- Notification was marked as unread (read_at changed from a value back to NULL)
    IF OLD.read_at IS NOT NULL AND NEW.read_at IS NULL THEN
        UPDATE notification_counters
        SET unread_count    = unread_count + 1,
            last_updated_at = NOW()
        WHERE user_id = NEW.user_id;
    END IF;
END$$
DELIMITER ;
```

### Trigger 3: Decrement when notification is deleted while unread

When `deleted_at` is set and the notification was still unread, remove it from the count.

```sql
DELIMITER $$
CREATE TRIGGER trg_decrement_counter_on_delete
AFTER UPDATE ON user_notifications
FOR EACH ROW
BEGIN
    -- Notification was soft-deleted and was still unread
    IF OLD.deleted_at IS NULL AND NEW.deleted_at IS NOT NULL
       AND NEW.read_at IS NULL
    THEN
        UPDATE notification_counters
        SET unread_count    = GREATEST(unread_count - 1, 0),
            last_updated_at = NOW()
        WHERE user_id = NEW.user_id;
    END IF;
END$$
DELIMITER ;
```

---

## Reading the Counter

```sql
-- Fast badge count (PK lookup — always < 1ms)
SELECT unread_count
FROM notification_counters
WHERE user_id = :user_id;
```

---

## Demonstrating the Counter

### Step 1: Check initial state

```sql
SELECT u.username, nc.unread_count
FROM notification_counters nc
JOIN users u ON u.id = nc.user_id
ORDER BY u.id;
```

### Step 2: Deliver a new notification to Alice

```sql
-- Insert the notification event
INSERT INTO notifications (title, body, category_id)
VALUES ('Bob sent you a message', 'Hey Alice, are you free tonight?', 1);

-- Deliver to Alice — trigger fires automatically
INSERT INTO user_notifications (user_id, notification_id)
VALUES (1, LAST_INSERT_ID());
```

### Step 3: Verify Alice's counter incremented

```sql
SELECT unread_count FROM notification_counters WHERE user_id = 1;
-- Should be 1 higher than before
```

### Step 4: Mark Alice's notification as read

```sql
UPDATE user_notifications
SET read_at = NOW()
WHERE user_id = 1 AND notification_id = (
    SELECT id FROM notifications WHERE title = 'Bob sent you a message'
);
```

### Step 5: Verify counter decremented

```sql
SELECT unread_count FROM notification_counters WHERE user_id = 1;
-- Back to the original count
```

---

## Counter Reconciliation (Periodic Correction)

Over time, edge cases (application bugs, failed transactions, direct SQL manipulation) can cause drift between the counter and the real unread count. Run a periodic job to reconcile:

```sql
-- Reconciliation query: fix any drifted counters
UPDATE notification_counters nc
JOIN (
    SELECT
        user_id,
        COUNT(*) AS real_count
    FROM user_notifications
    WHERE read_at    IS NULL
      AND deleted_at IS NULL
    GROUP BY user_id
) real ON real.user_id = nc.user_id
SET nc.unread_count    = real.real_count,
    nc.last_updated_at = NOW()
WHERE nc.unread_count != real.real_count;  -- only update drifted rows
```

Schedule this as a nightly job (or more frequently during testing).

---

## Counter vs Real COUNT Comparison

```sql
-- Audit: show where counter and real count disagree
SELECT
    u.username,
    nc.unread_count                                            AS cached_count,
    COUNT(un.id)                                               AS real_count,
    nc.unread_count - COUNT(COALESCE(un.id, 0))               AS drift
FROM notification_counters nc
JOIN users u ON u.id = nc.user_id
LEFT JOIN user_notifications un
    ON un.user_id    = nc.user_id
   AND un.read_at    IS NULL
   AND un.deleted_at IS NULL
GROUP BY nc.user_id, u.username, nc.unread_count
HAVING drift != 0;
```

---

## Full Schema Summary

By the end of this module, the complete notification system schema looks like:

```
users
  └─► user_notification_preferences  (user × category opt-in/out)
  └─► user_notifications              (inbox: user × notification delivery)
        └── notification_id ──► notifications  (the event record)
                                   └── category_id ──► notification_categories
  └─► notification_counters           (cached unread badge count)
```

---

## Best Practices

- Read the counter with a PK lookup (`WHERE user_id = ?`) — never `COUNT(*)` for the badge number.
- Use `GREATEST(unread_count - 1, 0)` in all decrements to guard against going negative from race conditions.
- Run a nightly reconciliation job — even well-written systems have occasional drift.
- On "mark all read", set the counter to `0` directly rather than decrementing in a loop.

## Common Mistakes

- Decrementing the counter without checking `IF OLD.read_at IS NULL` — marks something already-read causes a double decrement.
- Forgetting to decrement when a notification is soft-deleted while unread — counter shows higher than reality.
- Not initialising counter rows for new users — the first notification insert trigger's `ON DUPLICATE KEY UPDATE` handles new users, but a missing row causes confusion in reporting queries.

---

## Module Complete

You have built a complete, production-ready notification system database:

| File | Topic | Key Table(s) |
|------|-------|-------------|
| `1-intro.md` | Overview and architecture | — |
| `2-requirements.md` | Requirements | — |
| `3-setup.md` | Setup | `users` |
| `4-categories.md` | Categories and routing | `notification_categories` |
| `5-preferences.md` | Preferences | `user_notification_preferences` |
| `6-inbox.md` | Notifications and inbox | `notifications`, `user_notifications` |
| `7-read-status.md` | Read status tracking | `user_notifications.read_at` |
| `8-counter.md` | Unread badge counter | `notification_counters` |

## Next Step

Continue to [05-database-design-multi-languages](../05-database-design-multi-languages/) for internationalisation database design patterns.