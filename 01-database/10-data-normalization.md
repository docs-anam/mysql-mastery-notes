# Data Normalization

## What is Normalization?

**Normalization** is the process of organizing data in a database to:
- ✅ Eliminate redundancy (duplicate data)
- ✅ Minimize inconsistencies
- ✅ Improve data integrity
- ✅ Ensure referential integrity
- ✅ Organize data logically

## Why Normalize?

### Without Normalization (BAD DESIGN)

```
STUDENT_COURSE Table (Flat structure)
┌──────────┬──────────┬─────────────┬─────────────────────────────┐
│student_id│name      │course_ids   │course_names                 │
├──────────┼──────────┼─────────────┼─────────────────────────────┤
│ 1        │ Alice    │ 101,102,103 │ Math,Physics,Chemistry      │
│ 2        │ Bob      │ 101,104     │ Math,Biology                │
└──────────┴──────────┴─────────────┴─────────────────────────────┘

Problems:
❌ Redundant data (course_names repeated)
❌ Hard to query (parsing comma-separated values)
❌ Data anomalies (what if student not taking any course?)
❌ Update problems (change course name = multiple updates)
❌ Delete problems (delete course = delete student record!)
```

### With Normalization (GOOD DESIGN)

```
STUDENT Table
┌──────────┬──────────┐
│student_id│name      │
├──────────┼──────────┤
│ 1        │ Alice    │
│ 2        │ Bob      │
└──────────┴──────────┘

COURSE Table
┌───────────┬──────────┐
│course_id  │name      │
├───────────┼──────────┤
│ 101       │ Math     │
│ 102       │ Physics  │
│ 103       │ Chemistry│
│ 104       │ Biology  │
└───────────┴──────────┘

ENROLLMENT Table (Junction table)
┌──────────┬───────────┐
│student_id│course_id  │
├──────────┼───────────┤
│ 1        │ 101       │
│ 1        │ 102       │
│ 1        │ 103       │
│ 2        │ 101       │
│ 2        │ 104       │
└──────────┴───────────┘

Benefits:
✅ No data redundancy
✅ Easy to query
✅ Consistent data
✅ Update course name in one place
✅ Delete course without affecting students
```

## Normal Forms

Normalization follows a series of rules called **Normal Forms**. Each form builds on the previous.

### Normal Form Progression

```
Unnormalized Data
      ↓
   1NF (First Normal Form)
      ↓
   2NF (Second Normal Form)
      ↓
   3NF (Third Normal Form)
      ↓
  BCNF (Boyce-Codd Normal Form)
      ↓
   4NF, 5NF... (rarely needed)
```

## Dependency Concepts

### Functional Dependency

An attribute is **functionally dependent** on another if its value is determined by that other attribute.

```
student_id → name
(If you know student_id, you know the name)

employee_id → employee_name
(If you know employee_id, you know the name)
```

### Transitive Dependency

Attribute C is transitively dependent on A if:
- A → B (A determines B)
- B → C (B determines C)
- Therefore A → C

```
student_id → course_id → course_name
(student determines course, course determines name)
```

### Partial Dependency

An attribute depends on part of a composite key, not the whole key.

```
(student_id, course_id) → course_name
Course name depends only on course_id, not student_id!
```

## 0NF: Unnormalized Data

**Definition**: Data in no particular form, often with repeating groups.

### Example: BAD

```
CUSTOMER Table
┌──────────────┬──────────┬─────────────────────────────┐
│customer_id   │name      │phone_numbers                │
├──────────────┼──────────┼─────────────────────────────┤
│ 1            │ Alice    │ 555-1234, 555-5678, 555-9012│
│ 2            │ Bob      │ 555-4444, 555-5555          │
└──────────────┴──────────┴─────────────────────────────┘

Problem: Multiple values in one column!
```

## Next Steps

The following documents cover:

- **[First Normal Form (1NF)](11-normal-form-1.md)** - Atomicity
- **[Second Normal Form (2NF)](12-normal-form-2.md)** - Dependencies
- **[Third Normal Form (3NF)](13-normal-form-3.md)** - Transitive Dependencies

Each form removes a different type of redundancy.

## Key Normalization Principles

### 1. Atomic Values (1NF)
- Each cell contains only one value
- No repeating groups
- No arrays or delimited lists

### 2. Full Functional Dependency (2NF)
- Every non-key column depends on the ENTIRE primary key
- No partial dependencies

### 3. No Transitive Dependency (3NF)
- Non-key columns don't depend on other non-key columns
- Data depends only on the key, not on other data

## Normalization Levels

| Form | Purpose | Rule |
|------|---------|------|
| 1NF | Atomic data | Remove repeating groups |
| 2NF | Dependencies | Remove partial dependencies |
| 3NF | Transitive | Remove transitive dependencies |
| BCNF | Strict | Stricter than 3NF |

## Denormalization

**Important**: Sometimes we intentionally violate normal forms for **performance**.

```
Normalized: Multiple joins required (slower)
Denormalized: Duplicate some data (faster queries, bigger storage)
```

This is covered in **[Denormalization](14-denormalization.md)**.

## Real-World Example

### UNNORMALIZED:
```
ORDER Table
┌──────────┬──────────┬─────────────────────────┐
│order_id  │customer  │items                    │
├──────────┼──────────┼─────────────────────────┤
│ 1        │ Alice    │ item1,item2,item3       │
└──────────┴──────────┴─────────────────────────┘
```

### NORMALIZED (1NF):
```
ORDER Table
┌──────────┬──────────┐
│order_id  │customer  │
├──────────┼──────────┤
│ 1        │ Alice    │
└──────────┴──────────┘

ORDER_ITEM Table
┌──────────┬──────────┐
│order_id  │item      │
├──────────┼──────────┤
│ 1        │ item1    │
│ 1        │ item2    │
│ 1        │ item3    │
└──────────┴──────────┘
```

## Benefits Summary

✅ **Data Integrity** - Accurate, consistent data
✅ **No Redundancy** - Store data once
✅ **Easier Updates** - Change data in one place
✅ **Better Performance** - In normalized queries
✅ **Scalability** - Design scales with data growth
✅ **Maintainability** - Clear, logical structure

## When NOT to Normalize

❌ **Read-heavy systems** - May need denormalization
❌ **Data warehouses** - Often denormalized for analytics
❌ **Caching layers** - Redis caches denormalized data
❌ **Perfect performance** - May trade normalization for speed

## Key Takeaways

✅ Normalization removes redundancy and inconsistency
✅ Normal forms: 1NF, 2NF, 3NF (and beyond)
✅ 3NF is usually sufficient for most applications
✅ Each form removes a different type of problem
✅ Sometimes intentional denormalization improves performance

## Next Steps

1. **[First Normal Form (1NF)](11-normal-form-1.md)** - Learn about atomic data
2. **[Second Normal Form (2NF)](12-normal-form-2.md)** - Learn about dependencies
3. **[Third Normal Form (3NF)](13-normal-form-3.md)** - Learn about transitive dependencies
