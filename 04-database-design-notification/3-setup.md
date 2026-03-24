# Preparation

## Create the Database

```sql
CREATE DATABASE IF NOT EXISTS notification_app
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE notification_app;
```

> We use `utf8mb4` throughout this module. Notification content (titles, bodies) can contain emoji and non-ASCII characters — `utf8mb4` is the only charset that handles all of them correctly.

---

## Create the Users Table

Every notification is delivered **to** a user. This is the anchor for the entire schema.

```sql
CREATE TABLE users (
    id            BIGINT UNSIGNED  NOT NULL AUTO_INCREMENT,
    username      VARCHAR(50)      NOT NULL,
    email         VARCHAR(255)     NOT NULL,
    display_name  VARCHAR(100)     NOT NULL,
    avatar_url    VARCHAR(512)     NULL,
    is_active     BOOLEAN          NOT NULL DEFAULT TRUE,
    created_at    DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at    DATETIME         NOT NULL DEFAULT CURRENT_TIMESTAMP
                                        ON UPDATE CURRENT_TIMESTAMP,

    PRIMARY KEY (id),
    UNIQUE KEY uq_users_username (username),
    UNIQUE KEY uq_users_email    (email),
    INDEX        idx_users_active (is_active)
) ENGINE=InnoDB;
```

### Design Decisions

| Decision | Reason |
|----------|--------|
| `BIGINT UNSIGNED` for PK | Future-proofs for large user bases; avoids negative IDs |
| `UNIQUE` on username + email | Prevents duplicate accounts at the database level |
| `is_active` flag + index | Notifications should not be delivered to deactivated accounts |
| `updated_at` with `ON UPDATE` | Tracks the last modification without application code |
| `avatar_url NULL` | Optional — not all users have avatars at registration |

---

## Seed Data

```sql
INSERT INTO users (username, email, display_name, avatar_url) VALUES
('alice',   'alice@example.com',   'Alice Santoso',  'https://cdn.example.com/avatars/alice.jpg'),
('bob',     'bob@example.com',     'Bob Prasetyo',   'https://cdn.example.com/avatars/bob.jpg'),
('charlie', 'charlie@example.com', 'Charlie Wijaya', NULL),
('diana',   'diana@example.com',   'Diana Kusuma',   'https://cdn.example.com/avatars/diana.jpg'),
('evan',    'evan@example.com',    'Evan Hidayat',   NULL);
```

---

## Verify Setup

```sql
-- Check the database exists
SHOW DATABASES LIKE 'notification_app';

-- Check the table structure
DESCRIBE users;

-- Verify seed data
SELECT id, username, display_name, is_active FROM users;
```

Expected output:

```
+----+---------+------------------+-----------+
| id | username| display_name     | is_active |
+----+---------+------------------+-----------+
|  1 | alice   | Alice Santoso    |         1 |
|  2 | bob     | Bob Prasetyo     |         1 |
|  3 | charlie | Charlie Wijaya   |         1 |
|  4 | diana   | Diana Kusuma     |         1 |
|  5 | evan    | Evan Hidayat     |         1 |
+----+---------+------------------+-----------+
```

---

## Schema Checklist

At the end of this module we will have the following tables. Tick them off as you create them:

| Table | Purpose | Created in |
|-------|---------|-----------|
| `users` | Application users | This file ✅ |
| `notification_categories` | Types of notifications | `4-categories.md` |
| `user_notification_preferences` | Per-user category opt-in/out | `5-preferences.md` |
| `notifications` | Notification event record | `6-inbox.md` |
| `user_notifications` | Per-user delivery / inbox | `6-inbox.md` |
| `notification_counters` | Cached unread badge count | `8-counter.md` |

---

## Next Step

Continue to [4-categories.md](4-categories.md) to design the notification categories.