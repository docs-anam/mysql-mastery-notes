# Second Normal Form (2NF)

## What is 2NF?

**Second Normal Form (2NF)** requires:
- ✅ Table must be in **1NF** (atomic values)
- ✅ Every non-key column must depend on the **ENTIRE primary key**
- ✅ No **partial dependencies**

## Partial Dependency

A **partial dependency** occurs when a non-key column depends on only part of a composite primary key.

### Example: Violation

```
ENROLLMENT Table (Primary Key: student_id, course_id)
┌──────────┬───────────┬──────────────┬──────────────┐
│student_id│course_id  │course_name   │instructor    │
├──────────┼───────────┼──────────────┼──────────────┤
│ 1        │ 101       │ Math         │ Dr. Smith    │
│ 1        │ 102       │ Physics      │ Dr. Jones    │
│ 2        │ 101       │ Math         │ Dr. Smith    │
└──────────┴───────────┴──────────────┴──────────────┘

Problems:
❌ course_name depends only on course_id, not the full key
❌ instructor depends only on course_id, not the full key
❌ course_name "Math" appears multiple times (redundancy!)
❌ If Dr. Smith changes, must update multiple rows
```

## The 2NF Rule

**Every non-primary-key column must depend on the ENTIRE primary key, not just part of it.**

## Converting to 2NF

### Solution: Decompose Tables

**STUDENT Table**
```
┌──────────┬──────────┐
│student_id│name      │
├──────────┼──────────┤
│ 1        │ Alice    │
│ 2        │ Bob      │
└──────────┴──────────┘
```

**COURSE Table**
```
┌───────────┬──────────────┬──────────────┐
│course_id  │course_name   │instructor    │
├───────────┼──────────────┼──────────────┤
│ 101       │ Math         │ Dr. Smith    │
│ 102       │ Physics      │ Dr. Jones    │
└───────────┴──────────────┴──────────────┘
```

**ENROLLMENT Table (2NF)**
```
┌──────────┬───────────┐
│student_id│course_id  │
├──────────┼───────────┤
│ 1        │ 101       │
│ 1        │ 102       │
│ 2        │ 101       │
└──────────┴───────────┘
```

✅ No redundancy
✅ Easy to update course_name (one place)
✅ Each column depends on entire primary key

## Real-World Example: Order Details

### VIOLATING 2NF (BAD)

```
ORDER_DETAIL Table (Primary Key: order_id, product_id)
┌──────────┬────────────┬──────────────┬──────────┬─────────┐
│order_id  │product_id  │product_name  │unit_price│quantity │
├──────────┼────────────┼──────────────┼──────────┼─────────┤
│ 1        │ 101        │ Laptop       │ 999.99   │ 1       │
│ 1        │ 102        │ Mouse        │ 25.99    │ 2       │
│ 2        │ 101        │ Laptop       │ 999.99   │ 1       │
│ 2        │ 103        │ Keyboard     │ 79.99    │ 1       │
└──────────┴────────────┴──────────────┴──────────┴─────────┘

Issues:
❌ product_name depends only on product_id
❌ unit_price depends only on product_id
❌ "Laptop" and 999.99 repeated
❌ Price change requires multiple updates
```

### CONVERTED TO 2NF (GOOD)

**PRODUCT Table**
```
┌────────────┬──────────────┬──────────┐
│product_id  │product_name  │unit_price│
├────────────┼──────────────┼──────────┤
│ 101        │ Laptop       │ 999.99   │
│ 102        │ Mouse        │ 25.99    │
│ 103        │ Keyboard     │ 79.99    │
└────────────┴──────────────┴──────────┘
```

**ORDER_ITEM Table (2NF)**
```
┌──────────┬────────────┬──────────┐
│order_id  │product_id  │quantity  │
├──────────┼────────────┼──────────┤
│ 1        │ 101        │ 1        │
│ 1        │ 102        │ 2        │
│ 2        │ 101        │ 1        │
│ 2        │ 103        │ 1        │
└──────────┴────────────┴──────────┘
```

## SQL Implementation

### VIOLATING 2NF

```sql
CREATE TABLE order_detail_bad (
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  product_name VARCHAR(100),  -- Depends only on product_id!
  unit_price DECIMAL(10, 2),  -- Depends only on product_id!
  quantity INT,
  PRIMARY KEY (order_id, product_id)
);
```

### FOLLOWING 2NF

```sql
CREATE TABLE product (
  product_id INT PRIMARY KEY AUTO_INCREMENT,
  product_name VARCHAR(100) NOT NULL,
  unit_price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_item (
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (product_id) REFERENCES product(product_id)
);
```

## Dependency Analysis

### Check Each Column

For **ENROLLMENT** table with Primary Key (student_id, course_id):

| Column | Depends on | 2NF OK? |
|--------|-----------|--------|
| course_name | course_id only | ❌ NO |
| instructor | course_id only | ❌ NO |

### Fix by Moving to COURSE Table

Now course_name and instructor depend on course_id which is a PRIMARY KEY.

## When 2NF Matters

2NF becomes important with **composite primary keys**:

### Example: Student Enrollment
```
TABLE: enrollment (student_id, course_id)
- course_name should move to course table
- instructor should move to course table
```

### Example: Order Details
```
TABLE: order_item (order_id, product_id)
- product_name should move to product table
- unit_price should move to product table
```

### Example: Not Applicable
```
TABLE: student (student_id)
- Only one primary key column
- 2NF is automatically satisfied if 1NF is satisfied
```

## Data Redundancy Reduction

### BEFORE 2NF
```
OrderID  ProductID  ProductName  Quantity
1        101        Laptop       1
1        102        Mouse        2
2        101        Laptop       1
3        101        Laptop       1
4        102        Mouse        1

"Laptop" appears 3 times (redundancy!)
```

### AFTER 2NF
```
Product table:
ProductID  ProductName
101        Laptop
102        Mouse

OrderItem table:
OrderID  ProductID  Quantity
1        101        1
1        102        2
2        101        1
3        101        1
4        102        1

"Laptop" appears once (no redundancy!)
```

## Benefits of 2NF

✅ **Reduces redundancy** - Data stored once
✅ **Easier updates** - Change product_name in one place
✅ **Prevents anomalies** - No partial updates
✅ **Better consistency** - Single source of truth
✅ **Smaller tables** - Faster queries on each table

## Common 2NF Violations

| Pattern | Fix |
|---------|-----|
| Composite key with partial dependency | Move dependent column to new table |
| Non-key data depends on part of key | Extract to separate table |
| Repeating data in multiple rows | Move to reference table |

## Common Mistakes

❌ **Not recognizing composite keys** - Only applies to composite keys
❌ **Confusing with transitive dependency** - That's 3NF
❌ **Moving columns unnecessarily** - Only move if partial dependency exists
❌ **Forgetting to establish relationships** - Use foreign keys

## Tables with Single Primary Keys

If your table has **only one primary key column**, 2NF is automatically satisfied!

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY,  -- Single PK
  name VARCHAR(100),
  email VARCHAR(100)
);
-- All columns depend on customer_id (the entire key)
-- 2NF automatically satisfied if 1NF is satisfied
```

## Key Takeaways

✅ 2NF requires being in 1NF first
✅ No partial dependencies allowed
✅ Every non-key column must depend on ENTIRE primary key
✅ Mainly applies to tables with composite keys
✅ Fix by moving partially dependent columns to new tables

## Next Step

Learn about **[Third Normal Form (3NF)](13-normal-form-3.md)** - Removing transitive dependencies.
