# One-to-One Relationship

## Definition

Each row in table A relates to exactly one row in table B.

## Example Use Case

A `users` table and a `user_profiles` table where profile details are separated.

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(120) NOT NULL UNIQUE
);

CREATE TABLE user_profiles (
  user_id BIGINT PRIMARY KEY,
  full_name VARCHAR(120) NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## Next Step

Continue to [36-one-to-many-relationship.md](36-one-to-many-relationship.md).
