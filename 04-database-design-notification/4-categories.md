# Notification Categories

## Why Categories Exist

Categories serve three purposes:

1. **Routing** — the application decides which users to notify based on category preferences
2. **Filtering** — users can view their inbox filtered by category
3. **User control** — users can opt out of certain categories (see `4-user.md`)

---

## Create the Categories Table

```sql
CREATE TABLE notification_categories (
    id          INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100)  NOT NULL,
    slug        VARCHAR(50)   NOT NULL,
    description VARCHAR(255)  NULL,
    icon        VARCHAR(50)   NULL,
    is_required BOOLEAN       NOT NULL DEFAULT FALSE,
    is_active   BOOLEAN       NOT NULL DEFAULT TRUE,
    sort_order  TINYINT       NOT NULL DEFAULT 0,

    PRIMARY KEY (id),
    UNIQUE KEY uq_category_slug (slug),
    INDEX idx_category_active   (is_active)
) ENGINE=InnoDB;
```

### Column Guide

| Column | Purpose |
|--------|---------|
| `slug` | URL-safe identifier used in API endpoints and application code; never changes |
| `name` | Human-readable label displayed in the UI |
| `icon` | Icon class or emoji for UI display (e.g., `fa-envelope`, `🔔`) |
| `is_required` | `TRUE` for `system` — cannot be opted out of |
| `is_active` | `FALSE` soft-disables a category without deleting it |
| `sort_order` | Controls display order in the preferences UI |

### Why `slug` and Not Just `name`?

Application code and API endpoints reference categories by slug (`/notifications?category=social`), not by display name. Names can be changed or translated; slugs are stable identifiers.

---

## Seed Data

```sql
INSERT INTO notification_categories
    (id, name, slug, description, icon, is_required, is_active, sort_order)
VALUES
(1, 'Social',        'social',        'Likes, follows, mentions, comments',              '👥', FALSE, TRUE,  1),
(2, 'Transactional', 'transactional', 'Order updates, payment receipts, shipping alerts','📦', FALSE, TRUE,  2),
(3, 'Marketing',     'marketing',     'Promotions, sales, and product recommendations',  '🎉', FALSE, TRUE,  3),
(4, 'System',        'system',        'Maintenance windows, security alerts, policies',  '⚙️', TRUE,  TRUE,  4);
```

---

## Querying Categories

### List all active categories ordered for the preferences UI

```sql
SELECT id, name, slug, icon, is_required
FROM notification_categories
WHERE is_active = TRUE
ORDER BY sort_order;
```

### Get a category by slug (common in application routing)

```sql
SELECT * FROM notification_categories WHERE slug = 'transactional';
```

### Build a preferences page for user 1 (Alice)

```sql
SELECT
    c.id,
    c.name,
    c.slug,
    c.icon,
    c.is_required,
    COALESCE(p.is_enabled, TRUE) AS is_enabled
FROM notification_categories c
LEFT JOIN user_notification_preferences p
    ON p.category_id = c.id AND p.user_id = 1
WHERE c.is_active = TRUE
ORDER BY c.sort_order;
```

Result:

```
+----+--------------+---------------+----+-------------+------------+
| id | name         | slug          |icon| is_required | is_enabled |
+----+--------------+---------------+----+-------------+------------+
|  1 | Social       | social        | 👥 |           0 |          1 |
|  2 | Transactional| transactional | 📦 |           0 |          1 |
|  3 | Marketing    | marketing     | 🎉 |           0 |          0 |  ← Alice opted out
|  4 | System       | system        | ⚙️ |           1 |          1 |
+----+--------------+---------------+----+-------------+------------+
```

---

## Notification Routing Logic

When the application creates a notification, it must:

1. Look up the category by slug
2. Get all users who should receive it (target users)
3. Exclude users who have opted out (`is_enabled = FALSE` for that category)
4. Insert one `user_notifications` row per remaining user

```sql
-- Step 3: find users who have opted out of a category
-- (and should be excluded from receiving)
SELECT user_id
FROM user_notification_preferences
WHERE category_id = 3        -- marketing
  AND is_enabled  = FALSE;

-- Step: insert for all active users who have NOT opted out
INSERT INTO user_notifications (user_id, notification_id)
SELECT u.id, 5   -- notification_id = 5 (the new notification)
FROM users u
WHERE u.is_active = TRUE
  AND u.id NOT IN (
      SELECT user_id
      FROM user_notification_preferences
      WHERE category_id = 3 AND is_enabled = FALSE
  );
```

> For large user bases this operation is done asynchronously (message queue / background job), not inline with the HTTP request.

---

## Deactivating a Category

When a category is retired (e.g., you discontinue a marketing programme):

```sql
-- Soft-disable the category; existing notifications are preserved
UPDATE notification_categories
SET is_active = FALSE
WHERE slug = 'marketing';

-- Existing user_notifications rows remain; historical data is intact
-- New notifications in this category will not be created (application enforces this)
```

---

## Best Practices

- Use `slug` (not `id`) in application code to reference categories — it is human-readable and survives table recreation.
- Keep categories broad (4–8 total). Fine-grained sub-categories are better handled by the `data` JSON payload, not by multiplying table rows.
- Never hard-delete a category — set `is_active = FALSE`. Deleting cascades to preferences and historical notifications.

## Common Mistakes

- Using free-text category names directly in `user_notifications` — creates inconsistent strings across rows ("Marketing", "marketing", "MKTG").
- Not having a `system`/required category with `is_required = TRUE` — security alerts and maintenance notices must always reach users.
- Changing slugs after launch — client apps that filter by slug will break silently.

## Next Step

Continue to [5-preferences.md](5-preferences.md) to design user notification preferences.