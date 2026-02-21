# Third Normal Form (3NF)

## What is 3NF?

**Third Normal Form (3NF)** requires:
- ✅ Table must be in **2NF** (no partial dependencies)
- ✅ No **transitive dependencies**
- ✅ Non-key columns depend only on primary key, not on other non-key columns

## Transitive Dependency

A **transitive dependency** exists when:
- Column A is the primary key
- Column B depends on A
- Column C depends on B (not directly on A)
- Therefore A → B → C

### Example: Violation

```
STUDENT Table (Primary Key: student_id)
┌──────────┬──────────┬─────────────┬──────────────────┐
│student_id│name      │dept_id      │dept_name         │
├──────────┼──────────┼─────────────┼──────────────────┤
│ 1        │ Alice    │ 10          │ Computer Science │
│ 2        │ Bob      │ 10          │ Computer Science │
│ 3        │ Charlie  │ 20          │ Mathematics      │
└──────────┴──────────┴─────────────┴──────────────────┘

Dependency Chain:
student_id → dept_id → dept_name

Problem:
❌ dept_name depends on dept_id, not student_id
❌ dept_name appears multiple times (redundancy!)
❌ If department name changes, must update multiple rows
```

## The 3NF Rule

**Non-key columns must depend on the primary key, and ONLY the primary key.**

**Translation**: Non-key columns should NOT depend on other non-key columns.

## Converting to 3NF

### Solution: Create Department Table

**DEPARTMENT Table**
```
┌────────────┬──────────────────┐
│dept_id     │dept_name         │
├────────────┼──────────────────┤
│ 10         │ Computer Science │
│ 20         │ Mathematics      │
└────────────┴──────────────────┘
```

**STUDENT Table (3NF)**
```
┌──────────┬──────────┬─────────────┐
│student_id│name      │dept_id      │
├──────────┼──────────┼─────────────┤
│ 1        │ Alice    │ 10          │
│ 2        │ Bob      │ 10          │
│ 3        │ Charlie  │ 20          │
└──────────┴──────────┴─────────────┘
```

✅ No redundancy
✅ Easy to update department names (one place)
✅ Non-key columns depend only on primary key

## Real-World Examples

### Example 1: Employee and Company

#### VIOLATING 3NF (BAD)

```
EMPLOYEE Table
┌────────┬──────────┬──────────────┬─────────────┬───────────────────┐
│emp_id  │name      │company_id    │company_name │company_location   │
├────────┼──────────┼──────────────┼─────────────┼───────────────────┤
│ 1      │ Alice    │ 100          │ TechCorp    │ San Francisco     │
│ 2      │ Bob      │ 100          │ TechCorp    │ San Francisco     │
│ 3      │ Charlie  │ 200          │ DataInc     │ New York          │
└────────┴──────────┴──────────────┴─────────────┴───────────────────┘

Transitive Dependency:
emp_id → company_id → company_name
emp_id → company_id → company_location

Problems:
❌ "TechCorp" and "San Francisco" repeated
❌ If company moves, must update multiple rows
```

#### CONVERTED TO 3NF (GOOD)

**COMPANY Table**
```
┌──────────────┬──────────────┬───────────────────┐
│company_id    │company_name  │company_location   │
├──────────────┼──────────────┼───────────────────┤
│ 100          │ TechCorp     │ San Francisco     │
│ 200          │ DataInc      │ New York          │
└──────────────┴──────────────┴───────────────────┘
```

**EMPLOYEE Table (3NF)**
```
┌────────┬──────────┬──────────────┐
│emp_id  │name      │company_id    │
├────────┼──────────┼──────────────┤
│ 1      │ Alice    │ 100          │
│ 2      │ Bob      │ 100          │
│ 3      │ Charlie  │ 200          │
└────────┴──────────┴──────────────┘
```

### Example 2: Product and Category

#### VIOLATING 3NF (BAD)

```
PRODUCT Table
┌────────────┬──────────┬────────────┬──────────────────────┐
│product_id  │name      │category_id │category_description │
├────────────┼──────────┼────────────┼──────────────────────┤
│ 1          │ Laptop   │ 10         │ Electronics          │
│ 2          │ Monitor  │ 10         │ Electronics          │
│ 3          │ Desk     │ 20         │ Furniture            │
└────────────┴──────────┴────────────┴──────────────────────┘

Transitive Dependency:
product_id → category_id → category_description
```

#### CONVERTED TO 3NF (GOOD)

**CATEGORY Table**
```
┌────────────┬──────────────────────┐
│category_id │category_description  │
├────────────┼──────────────────────┤
│ 10         │ Electronics          │
│ 20         │ Furniture            │
└────────────┴──────────────────────┘
```

**PRODUCT Table (3NF)**
```
┌────────────┬──────────┬────────────┐
│product_id  │name      │category_id │
├────────────┼──────────┼────────────┤
│ 1          │ Laptop   │ 10         │
│ 2          │ Monitor  │ 10         │
│ 3          │ Desk     │ 20         │
└────────────┴──────────┴────────────┘
```

## SQL Implementation

### VIOLATING 3NF

```sql
CREATE TABLE employee_bad (
  emp_id INT PRIMARY KEY,
  name VARCHAR(100),
  company_id INT,
  company_name VARCHAR(100),      -- Depends on company_id!
  company_location VARCHAR(100)   -- Depends on company_id!
);
```

### FOLLOWING 3NF

```sql
CREATE TABLE company (
  company_id INT PRIMARY KEY AUTO_INCREMENT,
  company_name VARCHAR(100) NOT NULL,
  company_location VARCHAR(100)
);

CREATE TABLE employee (
  emp_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  company_id INT NOT NULL,
  FOREIGN KEY (company_id) REFERENCES company(company_id)
);
```

## Dependency Analysis

### Check Dependencies

```
EMPLOYEE table (Primary Key: emp_id)

Column           | Depends on  | Type
─────────────────┼─────────────┼─────────────
name             | emp_id      | Direct (OK)
company_id       | emp_id      | Direct (OK)
company_name     | company_id  | Transitive (BAD!)
company_location | company_id  | Transitive (BAD!)
```

### Fix by Creating Reference Table

Move company_name and company_location to separate COMPANY table:
```
COMPANY table
- company_id (PK)
- company_name (depends on company_id ✓)
- company_location (depends on company_id ✓)

EMPLOYEE table
- emp_id (PK)
- name (depends on emp_id ✓)
- company_id (FK, depends on emp_id ✓)
```

## Data Redundancy Reduction

### BEFORE 3NF
```
EmployeeID  Name      CompanyID  CompanyName  Location
1           Alice     100        TechCorp     SF
2           Bob       100        TechCorp     SF
3           Charlie   200        DataInc      NY
4           David     100        TechCorp     SF

"TechCorp" appears 3 times (redundancy!)
"SF" appears 3 times (redundancy!)
```

### AFTER 3NF
```
Company table:
CompanyID  CompanyName  Location
100        TechCorp     SF
200        DataInc      NY

Employee table:
EmployeeID  Name      CompanyID
1           Alice     100
2           Bob       100
3           Charlie   200
4           David     100

"TechCorp" appears once (no redundancy!)
```

## Benefits of 3NF

✅ **Eliminates redundancy** - No repeating data
✅ **Prevents update anomalies** - Change once, affects all
✅ **Easier maintenance** - Clear data relationships
✅ **Better performance** - Smaller, focused tables
✅ **Improved consistency** - Single source of truth

## 3NF vs 2NF vs 1NF

| Normal Form | Requirement | Eliminates |
|-------------|-------------|-----------|
| 1NF | Atomic values | Repeating groups |
| 2NF | No partial dependencies | Composite key issues |
| 3NF | No transitive dependencies | Non-key dependencies |

## Practical Guideline

**Is this a 3NF question?**

Ask: "Does a non-key column depend on another non-key column?"

- **YES** → It's a transitive dependency → Not 3NF
- **NO** → It's 3NF (assuming 1NF and 2NF)

## Common Patterns Requiring 3NF

### Pattern 1: Description Data
```
Table has: ID, Name, Description
Move Description if it describes something that has an ID
```

### Pattern 2: Reference Data
```
Table has: ID, CategoryID, CategoryName
Move CategoryName to Category table if it's just a label for CategoryID
```

### Pattern 3: Status/Type Information
```
Table has: ID, Status, StatusDescription
Move StatusDescription to Status table
```

## Complete Example: School Database

### BEFORE (Multiple Violations)

```sql
CREATE TABLE student_bad (
  student_id INT PRIMARY KEY,
  name VARCHAR(100),
  dept_id INT,
  dept_name VARCHAR(100),         -- 3NF violation!
  dept_building VARCHAR(100),     -- 3NF violation!
  advisor_id INT,
  advisor_name VARCHAR(100)       -- 3NF violation!
);
```

### AFTER (3NF Compliant)

```sql
CREATE TABLE department (
  dept_id INT PRIMARY KEY,
  dept_name VARCHAR(100),
  dept_building VARCHAR(100)
);

CREATE TABLE advisor (
  advisor_id INT PRIMARY KEY,
  advisor_name VARCHAR(100),
  dept_id INT,
  FOREIGN KEY (dept_id) REFERENCES department(dept_id)
);

CREATE TABLE student (
  student_id INT PRIMARY KEY,
  name VARCHAR(100),
  dept_id INT,
  advisor_id INT,
  FOREIGN KEY (dept_id) REFERENCES department(dept_id),
  FOREIGN KEY (advisor_id) REFERENCES advisor(advisor_id)
);
```

## Key Takeaways

✅ 3NF requires being in 2NF first
✅ No transitive dependencies allowed
✅ Non-key columns depend on primary key only
✅ Fix by creating separate tables for dependent data
✅ 3NF is usually sufficient for most applications

## When NOT to Enforce 3NF

Sometimes for performance reasons:
- **Read-heavy queries** benefit from denormalization
- **Reporting systems** often store denormalized copies
- **Caches** intentionally duplicate data
- See **[Denormalization](14-denormalization.md)** for when and why

## Next Step

Learn about **[Denormalization](14-denormalization.md)** - When to intentionally break these rules.
