# Many-to-Many Relationship

## Definition

Rows in table A can relate to many rows in table B and vice versa.

## Modeling Pattern

Use a bridge table (junction table) that stores both foreign keys.

```sql
CREATE TABLE student_courses (
  student_id BIGINT NOT NULL,
  course_id BIGINT NOT NULL,
  enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (student_id, course_id)
);
```

## Next Step

Continue to [38-join-types.md](38-join-types.md).
