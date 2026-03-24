# User Notification Preferences

## The Problem

Not all users want all notifications. A user might want:
- ✅ Order shipping updates (transactional)
- ✅ Direct messages (social)
- ❌ Marketing promotions
- ❌ Weekly digest emails

The database must support **per-user, per-category opt-in/out** without polluting the `users` table with dozens of boolean columns.

---

## Design Options

### Option A: Extra Columns on the Users Table

```sql
-- BAD: does not scale to new categories
ALTER TABLE users ADD COLUMN notify_social     BOOLEAN DEFAULT TRUE;
ALTER TABLE users ADD COLUMN notify_marketing  BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN notify_transact   BOOLEAN DEFAULT TRUE;
-- ...add a column every time a category is created
```

**Problem**: Adding a new notification category requires a schema migration. This is an `ALTER TABLE` on what could be a very large users table in production.

### Option B: JSON Column

```sql
-- ACCEPTABLE but hard to query
ALTER TABLE users ADD COLUMN notification_prefs JSON;
-- {"social": true, "marketing": false}
```

**Problem**: Cannot use a simple `WHERE` clause or index on a JSON key. Checking preferences at query time is expensive.

### Option C: Separate Preferences Table (Chosen)

A normalised approach where each preference is its own row:

```
users (1 row)  →  user_notification_preferences (N rows, one per category)
```

This scales to any number of categories without schema changes.

---

## Create the Preferences Table

> Note: `notification_categories` must exist before this table (created in `4-categories.md`).

```sql
CREATE TABLE user_notification_preferences (
    id          BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    user_id     BIGINT UNSIGNED  NOT NULL,
    category_id INT UNSIGNED     NOT NULL,
    is_enabled  BOOLEAN          NOT NULL DEFAULT TRUE,
    updated_at  DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP
                                      ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_user_category  (user_id, category_id),
    CONSTRAINT fk_unp_user
        FOREIGN KEY (user_id)     REFERENCES users (id)
        ON DELETE CASCADE,
    CONSTRAINT fk_unp_category
        FOREIGN KEY (category_id) REFERENCES notification_categories (id)
        ON DELETE CASCADE
) ENGINE=InnoDB;
```

### Why `UNIQUE KEY uq_user_category`?

Each user can have **at most one** preference row per category. Without this, an application bug could insert duplicate rows and create ambiguity about whether a user has opted in or out.

---

## Opt-In vs Opt-Out Default

There are two philosophical approaches:

| Strategy | Behaviour | Row stored when? |
|----------|-----------|-----------------|
| **Opt-out (default on)** | User receives notifications unless they explicitly disable | Only when user disables a category |
| **Opt-in (default off)** | User receives nothing unless they explicitly enable | Only when user enables a category |

This system uses **opt-out by default** for most categories — if no preference row exists for a user/category pair, we assume `is_enabled = TRUE`. This is common for transactional and system notifications.

```sql
-- Check if a user wants a category's notifications
-- Returns TRUE if:
--   a) No preference row exists (default on), OR
--   b) Preference row exists with is_enabled = TRUE

SELECT
    u.id,
    u.username,
    c.slug AS category,
    COALESCE(p.is_enabled, TRUE) AS will_receive
FROM users u
CROSS JOIN notification_categories c
LEFT JOIN user_notification_preferences p
    ON p.user_id = u.id AND p.category_id = c.id
WHERE u.id = 1;
```

---

## Seeding Preferences

```sql
-- Assume notification_categories is already populated with IDs 1-4
-- category 3 = 'marketing'

-- Alice opts out of marketing
INSERT INTO user_notification_preferences (user_id, category_id, is_enabled)
VALUES (1, 3, FALSE);

-- Charlie opts out of marketing AND social
INSERT INTO user_notification_preferences (user_id, category_id, is_enabled)
VALUES (3, 3, FALSE),
       (3, 1, FALSE);
```

---

## Querying Preferences

### Get all preferences for a user

```sql
SELECT
    c.name        AS category,
    c.slug,
    COALESCE(p.is_enabled, TRUE) AS is_enabled
FROM notification_categories c
LEFT JOIN user_notification_preferences p
    ON p.category_id = c.id AND p.user_id = 1
ORDER BY c.name;
```

### Get all users who opted out of a category

```sql
SELECT u.username, u.email
FROM user_notification_preferences p
JOIN users u ON u.id = p.user_id
WHERE p.category_id = 3        -- marketing
  AND p.is_enabled = FALSE;
```

### Update a preference (upsert)

```sql
-- MySQL INSERT ... ON DUPLICATE KEY UPDATE is the clean upsert pattern
INSERT INTO user_notification_preferences (user_id, category_id, is_enabled)
VALUES (2, 3, FALSE)   -- Bob opts out of marketing
ON DUPLICATE KEY UPDATE
    is_enabled = VALUES(is_enabled),
    updated_at = NOW();
```

---

## Enforcing the System Category Rule

The `system` category must always be delivered — users cannot opt out. Enforce this in the application layer:

```sql
-- Application pseudocode: reject opt-out of 'system'
DELIMITER $$
CREATE TRIGGER trg_prevent_system_optout
BEFORE INSERT ON user_notification_preferences
FOR EACH ROW
BEGIN
    DECLARE v_slug VARCHAR(50);
    SELECT slug INTO v_slug
    FROM notification_categories WHERE id = NEW.category_id;
    IF v_slug = 'system' AND NEW.is_enabled = FALSE THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot opt out of system notifications';
    END IF;
END$$
DELIMITER ;
```

---

## Best Practices

- Use the `UPSERT` (`INSERT ... ON DUPLICATE KEY UPDATE`) pattern for preference updates to avoid duplicate row errors.
- Never delete preference rows — set `is_enabled = FALSE` instead. This preserves history and avoids re-creating rows.
- Cache preferences in application memory (Redis, etc.) to avoid hitting the database on every notification routing decision.

## Common Mistakes

- Adding preference columns directly to the users table — prevents adding new categories without schema migrations.
- Not putting a `UNIQUE` constraint on `(user_id, category_id)` — creates ambiguous duplicates.
- Treating a missing preference row as "opted out" — this inverts the UX default and causes users to miss notifications after a category is added.

## Next Step

Continue to [6-inbox.md](6-inbox.md) to design the core notifications and user inbox tables.