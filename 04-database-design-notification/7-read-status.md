# Read / Unread Status

## The Requirements

From our specification:
- Each notification in a user's inbox has an independent read status
- A user can mark a single notification as read
- A user can mark **all** notifications as read at once
- The system must record **when** a notification was read

---

## Design Approaches Compared

### Approach A: `is_read BOOLEAN`

```sql
ALTER TABLE user_notifications ADD COLUMN is_read BOOLEAN NOT NULL DEFAULT FALSE;
```

**Pros**: Simple `WHERE is_read = FALSE` query  
**Cons**: Throws away information — we don't know if something was read 2 minutes or 2 weeks ago. Cannot sort by "recently read".

---

### Approach B: `read_at DATETIME NULL` (Chosen)

```sql
-- Already present in our user_notifications schema
-- NULL  → unread
-- timestamp → read at this datetime
read_at DATETIME NULL
```

**Pros**:
- `NULL` means unread; `NOT NULL` means read — one column carries both state and timestamp
- `WHERE read_at IS NULL` is as fast as `WHERE is_read = FALSE`
- Enables "marked read on mobile at 9:31am" analytics

---

### Approach C: Separate `read_events` Table

```sql
CREATE TABLE read_events (
    user_id         BIGINT UNSIGNED NOT NULL,
    notification_id BIGINT UNSIGNED NOT NULL,
    read_at         DATETIME NOT NULL,
    PRIMARY KEY (user_id, notification_id)
);
```

**Pros**: Immutable audit trail; the main inbox table is never updated  
**Cons**: Every "is unread?" query requires a `LEFT JOIN`. Schema is more complex. Almost never worth it.

### Summary Table

| Approach | Unread query | Read timestamp | Complexity | Recommended? |
|---------|-------------|----------------|-----------|-------------|
| `is_read BOOLEAN` | Simple | ❌ No | Low | Only for throwaway projects |
| `read_at DATETIME NULL` | Simple | ✅ Yes | Low | ✅ Yes — standard pattern |
| Separate `read_events` table | `LEFT JOIN` needed | ✅ Yes | High | For audit-heavy systems only |

---

## Marking One Notification as Read

```sql
-- Mark a specific inbox item as read for a specific user
-- (always scope to user_id to prevent one user marking another's notification)
UPDATE user_notifications
SET read_at = NOW()
WHERE id      = :inbox_id      -- the user_notifications.id
  AND user_id = :user_id       -- security: user can only update their own rows
  AND read_at IS NULL;         -- idempotent: skip if already read
```

> The `read_at IS NULL` condition makes this **idempotent** — calling it twice has the same result as calling it once.

### What happens to the counter?

Marking one notification as read must also decrement the counter. This is handled by a trigger in `8-counter.md`. Applications using the stored procedure approach should call:

```sql
CALL mark_notification_read(:user_id, :inbox_id);
```

---

## Marking All Notifications as Read

```sql
UPDATE user_notifications
SET read_at = NOW()
WHERE user_id    = :user_id
  AND read_at    IS NULL       -- only touch currently unread
  AND deleted_at IS NULL;      -- skip deleted items
```

The counter is reset to 0 separately (see `8-counter.md`).

---

## Marking as Unread

Not in the original spec but commonly requested. Simply clear `read_at`:

```sql
UPDATE user_notifications
SET read_at = NULL
WHERE id      = :inbox_id
  AND user_id = :user_id;
```

> The unread counter trigger must increment on `read_at` going from non-NULL back to NULL (see `8-counter.md`).

---

## Soft Deleting a Notification

```sql
-- User dismisses a notification (soft delete)
UPDATE user_notifications
SET deleted_at = NOW()
WHERE id      = :inbox_id
  AND user_id = :user_id;
```

If the dismissed notification was unread, the counter must also be decremented. Wrap both in a transaction:

```sql
START TRANSACTION;

-- 1. Mark as deleted
UPDATE user_notifications
SET deleted_at = NOW()
WHERE id = :inbox_id AND user_id = :user_id;

-- 2. If it was unread, decrement the counter
UPDATE notification_counters
SET unread_count = GREATEST(unread_count - 1, 0)
WHERE user_id = :user_id
  AND (SELECT read_at FROM user_notifications WHERE id = :inbox_id) IS NULL;

COMMIT;
```

> `GREATEST(..., 0)` prevents the counter going negative if there is a race condition.

---

## Stored Procedure: mark_notification_read

A stored procedure is the safest way to ensure both the status update and counter decrement happen atomically.

```sql
DELIMITER $$
CREATE PROCEDURE mark_notification_read(
    IN p_user_id   BIGINT UNSIGNED,
    IN p_inbox_id  BIGINT UNSIGNED
)
BEGIN
    DECLARE v_already_read BOOLEAN;

    -- Check current read status
    SELECT (read_at IS NOT NULL) INTO v_already_read
    FROM user_notifications
    WHERE id = p_inbox_id AND user_id = p_user_id;

    IF v_already_read = FALSE THEN
        START TRANSACTION;

        UPDATE user_notifications
        SET read_at = NOW()
        WHERE id = p_inbox_id AND user_id = p_user_id;

        UPDATE notification_counters
        SET unread_count = GREATEST(unread_count - 1, 0),
            last_updated_at = NOW()
        WHERE user_id = p_user_id;

        COMMIT;
    END IF;
END$$
DELIMITER ;
```

Usage:

```sql
-- Bob (user_id=2) reads inbox item 1
CALL mark_notification_read(2, 1);
```

---

## Stored Procedure: mark_all_notifications_read

```sql
DELIMITER $$
CREATE PROCEDURE mark_all_notifications_read(
    IN p_user_id BIGINT UNSIGNED
)
BEGIN
    START TRANSACTION;

    UPDATE user_notifications
    SET read_at = NOW()
    WHERE user_id    = p_user_id
      AND read_at    IS NULL
      AND deleted_at IS NULL;

    UPDATE notification_counters
    SET unread_count    = 0,
        last_updated_at = NOW()
    WHERE user_id = p_user_id;

    COMMIT;
END$$
DELIMITER ;
```

---

## Verify Read Status

```sql
-- Check Bob's inbox with read status
SELECT
    un.id,
    n.title,
    CASE WHEN un.read_at IS NULL THEN 'unread' ELSE 'read' END AS status,
    un.read_at,
    un.created_at
FROM user_notifications un
JOIN notifications n ON n.id = un.notification_id
WHERE un.user_id    = 2
  AND un.deleted_at IS NULL
ORDER BY un.created_at DESC;
```

---

## Best Practices

- Always include `AND user_id = :user_id` when updating `user_notifications` — prevents users from marking other people's notifications as read (broken access control).
- Make mark-as-read **idempotent** by checking `read_at IS NULL` in the `WHERE` clause — safe to call multiple times.
- Use a stored procedure or transaction to keep the status update and counter decrement atomic.

## Common Mistakes

- Updating `user_notifications` without scoping by `user_id` — any authenticated user could mark anyone else's notifications as read.
- Not wrapping the status update and counter decrement in a transaction — a partial write leaves the counter wrong.
- Setting `read_at` to `NULL` in a hard delete instead of using `deleted_at` — loses audit trail and corrupts the counter.

## Next Step

Continue to [8-counter.md](8-counter.md) to implement the fast unread badge counter.