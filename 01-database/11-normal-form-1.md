# First Normal Form (1NF)

## What is 1NF?

**First Normal Form (1NF)** requires that:
- ✅ All column values are **atomic** (indivisible)
- ✅ No repeating groups
- ✅ No arrays or comma-separated lists in columns
- ✅ Each cell contains exactly one value

## The 1NF Rule

**Every attribute must contain only atomic (single) values.**

## Example: Violating 1NF

### BAD: Non-Atomic Data

```
STUDENT Table (NOT 1NF)
┌──────────┬─────────┬──────────────────────────┐
│student_id│name     │courses                   │
├──────────┼─────────┼──────────────────────────┤
│ 1        │ Alice   │ Math, Physics, Chemistry │
│ 2        │ Bob     │ Math, Biology            │
│ 3        │ Charlie │ History                  │
└──────────┴─────────┴──────────────────────────┘

Problems:
❌ courses column contains multiple values
❌ How to query "Who takes Math?"
❌ How to update one course without affecting others?
❌ Hard to enforce data consistency
```

## Converting to 1NF

### Solution 1: Create Separate Table

**STUDENT Table (1NF)**
```
┌──────────┬─────────┐
│student_id│name     │
├──────────┼─────────┤
│ 1        │ Alice   │
│ 2        │ Bob     │
│ 3        │ Charlie │
└──────────┴─────────┘
```

**ENROLLMENT Table (1NF)**
```
┌──────────┬──────────┐
│student_id│course    │
├──────────┼──────────┤
│ 1        │ Math     │
│ 1        │ Physics  │
│ 1        │ Chemistry│
│ 2        │ Math     │
│ 2        │ Biology  │
│ 3        │ History  │
└──────────┴──────────┘
```

✅ Now atomic (one value per cell)
✅ Easy to query
✅ Easy to update
✅ Easy to maintain consistency

## Real-World Examples of 1NF Violations

### Phone Numbers (BAD)

```
CUSTOMER Table (NOT 1NF)
┌──────────┬──────────┬──────────────────────────┐
│customer_id│name     │phone_numbers             │
├──────────┼──────────┼──────────────────────────┤
│ 1        │ Alice    │ 555-1234, 555-5678      │
│ 2        │ Bob      │ 555-4444                │
└──────────┴──────────┴──────────────────────────┘
```

### Fixed: 1NF

```
CUSTOMER Table
┌──────────┬──────────┐
│customer_id│name     │
├──────────┼──────────┤
│ 1        │ Alice    │
│ 2        │ Bob      │
└──────────┴──────────┘

PHONE Table
┌──────────┬──────────────┐
│customer_id│phone_number  │
├──────────┼──────────────┤
│ 1        │ 555-1234    │
│ 1        │ 555-5678    │
│ 2        │ 555-4444    │
└──────────┴──────────────┘
```

### Skills (BAD)

```
EMPLOYEE Table (NOT 1NF)
┌─────────┬──────────┬──────────────────────┐
│emp_id   │name      │skills                │
├─────────┼──────────┼──────────────────────┤
│ 1       │ Alice    │ Java, Python, SQL    │
│ 2       │ Bob      │ JavaScript, HTML     │
└─────────┴──────────┴──────────────────────┘
```

### Fixed: 1NF

```
EMPLOYEE Table
┌─────────┬──────────┐
│emp_id   │name      │
├─────────┼──────────┤
│ 1       │ Alice    │
│ 2       │ Bob      │
└─────────┴──────────┘

SKILL Table
┌─────────┬──────────┐
│emp_id   │skill     │
├─────────┼──────────┤
│ 1       │ Java     │
│ 1       │ Python   │
│ 1       │ SQL      │
│ 2       │ JavaScript
│ 2       │ HTML     │
└─────────┴──────────┘
```

## SQL Implementation

### Creating 1NF Tables

```sql
-- CUSTOMER Table
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL
);

-- PHONE Table (for multiple phone numbers)
CREATE TABLE phone (
  phone_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  phone_number VARCHAR(15) NOT NULL,
  phone_type ENUM('mobile', 'home', 'work'),
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);

-- EMPLOYEE Table
CREATE TABLE employee (
  emp_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL
);

-- SKILL Table (for multiple skills)
CREATE TABLE skill (
  skill_id INT PRIMARY KEY AUTO_INCREMENT,
  emp_id INT NOT NULL,
  skill_name VARCHAR(50) NOT NULL,
  proficiency_level ENUM('beginner', 'intermediate', 'expert'),
  FOREIGN KEY (emp_id) REFERENCES employee(emp_id)
);
```

## Querying 1NF Data

### Find customer with multiple phone numbers

```sql
SELECT c.customer_id, c.name, p.phone_number, p.phone_type
FROM customer c
JOIN phone p ON c.customer_id = p.customer_id
WHERE c.customer_id = 1;

Result:
┌──────────────┬───────┬──────────────┬────────────┐
│customer_id   │name   │phone_number  │phone_type  │
├──────────────┼───────┼──────────────┼────────────┤
│ 1            │ Alice │ 555-1234     │ mobile     │
│ 1            │ Alice │ 555-5678     │ home       │
└──────────────┴───────┴──────────────┴────────────┘
```

### Find employees with specific skill

```sql
SELECT e.emp_id, e.name, s.skill_name, s.proficiency_level
FROM employee e
JOIN skill s ON e.emp_id = s.emp_id
WHERE s.skill_name = 'Java';

Result:
┌────────┬──────────┬────────────┬───────────────────┐
│emp_id  │name      │skill_name  │proficiency_level  │
├────────┼──────────┼────────────┼───────────────────┤
│ 1      │ Alice    │ Java       │ expert            │
└────────┴──────────┴────────────┴───────────────────┘
```

## Benefits of 1NF

✅ **Simple to query** - Use standard SQL joins
✅ **Atomic data** - Each cell has one value
✅ **No parsing** - No comma-separated values to split
✅ **Easy updates** - Change one skill without affecting others
✅ **Consistent** - Enforced by database structure
✅ **Scalable** - Can add unlimited courses/skills

## Common 1NF Violations

| Type | Example | Fix |
|------|---------|-----|
| Repeating groups | Courses: Math, Physics | Separate table |
| Comma-separated | Tags: Tag1,Tag2,Tag3 | Separate table |
| Arrays | Items: [A, B, C] | Separate table |
| Lists | Related IDs: 1,2,3 | Separate table |
| Multiple values | Phones: 555-1234, 555-5678 | Separate table |

## Before and After Comparison

### BEFORE (NOT 1NF)
```sql
CREATE TABLE enrollment (
  student_id INT PRIMARY KEY,
  name VARCHAR(100),
  courses VARCHAR(255)  -- BAD: multiple values!
);

INSERT INTO enrollment VALUES 
  (1, 'Alice', 'Math, Physics, Chemistry'),
  (2, 'Bob', 'Math, Biology');
```

### AFTER (1NF)
```sql
CREATE TABLE student (
  student_id INT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE course (
  course_id INT PRIMARY KEY,
  course_name VARCHAR(100)
);

CREATE TABLE enrollment (
  student_id INT,
  course_id INT,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES student(student_id),
  FOREIGN KEY (course_id) REFERENCES course(course_id)
);

INSERT INTO enrollment VALUES 
  (1, 1), (1, 2), (1, 3), (2, 1), (2, 4);
```

## Key Takeaways

✅ 1NF = Atomic values only (one value per cell)
✅ No repeating groups
✅ No comma-separated or array data
✅ Create separate tables for multi-valued attributes
✅ Use junction/bridge tables for relationships

## When You See Violations

If you see:
- Comma-separated values: `'Java, Python, SQL'`
- Multiple values in one cell: `'555-1234, 555-5678'`
- Repeating columns: `course1, course2, course3`

**Solution**: Create a separate table!

## Next Step

Learn about **[Second Normal Form (2NF)](12-normal-form-2.md)** - Removing partial dependencies.
