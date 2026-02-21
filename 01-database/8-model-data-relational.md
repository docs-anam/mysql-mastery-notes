# Relational Model

## What is the Relational Model?

The **Relational Model** is a way of organizing data using **tables (relations)** where data is stored in rows and columns, connected through **keys**.

## Core Concepts

### 1. Relation (Table)

A **relation** is a table with:
- **Columns (Attributes)**: Represent properties
- **Rows (Tuples)**: Represent individual records
- **Primary Key**: Uniquely identifies each row

### Example: Customer Table

```
┌────────────┬─────────────┬──────────────────┬─────────────────┐
│ customer_id│    name     │      email       │      phone      │
├────────────┼─────────────┼──────────────────┼─────────────────┤
│     1      │    John     │ john@example.com │  555-1234       │
│     2      │    Jane     │ jane@example.com │  555-5678       │
│     3      │   Robert    │ bob@example.com  │  555-9012       │
└────────────┴─────────────┴──────────────────┴─────────────────┘

Table Name: CUSTOMER
Columns (Attributes): customer_id, name, email, phone
Rows (Tuples): 3 records/rows
Primary Key: customer_id
```

### 2. Domain

A **domain** is the set of allowed values for an attribute.

```
Attribute: name
Domain: VARCHAR(100) - any string up to 100 characters

Attribute: email
Domain: VARCHAR(100) - email format

Attribute: customer_id
Domain: INT - positive integers
```

### 3. Tuple (Row)

A single record in a table.

```
Example tuple from CUSTOMER table:
(customer_id: 1, name: "John", email: "john@example.com", phone: "555-1234")
```

## Relational Database Structure

### Tables and Relationships

```
TABLE: CUSTOMER
┌────────────┬──────────┐
│customer_id │ name     │
├────────────┼──────────┤
│ 1          │ John     │
│ 2          │ Jane     │
└────────────┴──────────┘
      │
      │ References via Foreign Key
      │
      ▼
TABLE: ORDERS
┌────────┬────────────┬──────────┐
│order_id│customer_id │date      │
├────────┼────────────┼──────────┤
│ 101    │ 1          │2024-02-01│
│ 102    │ 1          │2024-02-02│
│ 103    │ 2          │2024-02-03│
└────────┴────────────┴──────────┘
```

## Keys in Relational Model

### Primary Key
- Uniquely identifies each tuple (row)
- Every relation must have one
- Cannot be NULL
- Cannot contain duplicates

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY,  -- Primary Key
  name VARCHAR(100)
);
```

### Foreign Key
- References primary key in another table
- Creates relationships between relations
- Enforces referential integrity

```sql
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);
```

### Candidate Key
- Any attribute(s) that could uniquely identify a row
- Primary key is chosen from candidate keys

```
CUSTOMER table:
Candidate keys:
1. customer_id (chosen as PRIMARY KEY)
2. email (assuming each customer has unique email)
3. phone (assuming each customer has unique phone)
```

### Composite Key
- Primary key made of multiple attributes
- Used in junction tables

```sql
CREATE TABLE student_course (
  student_id INT NOT NULL,
  course_id INT NOT NULL,
  PRIMARY KEY (student_id, course_id)
);
```

## Relationships in Relational Model

### One-to-One (1:1)

```sql
CREATE TABLE person (
  person_id INT PRIMARY KEY
);

CREATE TABLE passport (
  passport_id INT PRIMARY KEY,
  person_id INT UNIQUE NOT NULL,
  FOREIGN KEY (person_id) REFERENCES person(person_id)
);

-- One person has one passport
-- Each passport belongs to one person
```

### One-to-Many (1:N)

```sql
CREATE TABLE author (
  author_id INT PRIMARY KEY
);

CREATE TABLE book (
  book_id INT PRIMARY KEY,
  author_id INT NOT NULL,
  FOREIGN KEY (author_id) REFERENCES author(author_id)
);

-- One author writes many books
-- Each book has one author
```

### Many-to-Many (M:N)

```sql
CREATE TABLE student (
  student_id INT PRIMARY KEY
);

CREATE TABLE course (
  course_id INT PRIMARY KEY
);

CREATE TABLE enrollment (
  student_id INT NOT NULL,
  course_id INT NOT NULL,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES student(student_id),
  FOREIGN KEY (course_id) REFERENCES course(course_id)
);

-- Many students enroll in many courses
-- Requires junction table (ENROLLMENT)
```

## Relational Integrity

### Referential Integrity

Data in one table must reference valid data in another table.

```sql
-- VALID: customer_id 1 exists
INSERT INTO orders (order_id, customer_id) VALUES (1, 1);  -- ✅

-- INVALID: customer_id 999 doesn't exist
INSERT INTO orders (order_id, customer_id) VALUES (2, 999);  -- ❌ ERROR!
```

### Entity Integrity

Every tuple must have a unique primary key.

```sql
INSERT INTO customer (customer_id, name) VALUES (1, 'John');     -- ✅
INSERT INTO customer (customer_id, name) VALUES (1, 'Jane');     -- ❌ Duplicate!
INSERT INTO customer (customer_id, name) VALUES (NULL, 'Bob');   -- ❌ NULL PK!
```

### Domain Integrity

Values must be within allowed domain.

```sql
CREATE TABLE product (
  product_id INT PRIMARY KEY,
  price DECIMAL(10, 2) CHECK (price > 0)
);

INSERT INTO product VALUES (1, 29.99);   -- ✅ Valid
INSERT INTO product VALUES (2, -10);    -- ❌ Fails CHECK constraint
INSERT INTO product VALUES (3, 'cheap');-- ❌ Wrong data type
```

## Relational Operations

### Selection (WHERE)

```sql
SELECT * FROM customer WHERE customer_id = 1;
-- Returns rows matching condition
```

### Projection (SELECT columns)

```sql
SELECT name, email FROM customer;
-- Returns specific columns
```

### Join (Combine tables)

```sql
SELECT c.name, o.order_id 
FROM customer c
JOIN orders o ON c.customer_id = o.customer_id;
-- Combines data from multiple tables
```

### Union (Combine result sets)

```sql
SELECT name FROM customer
UNION
SELECT product_name FROM product;
-- Combines results from multiple queries
```

## Real-World Example: Library System

### Entities and Relationships

```
AUTHOR ──[1:M]── BOOK ──[M:N]── MEMBER
                   │
                   │ [1:M]
                   │
                   ▼
              LOAN
```

### SQL Implementation

```sql
CREATE TABLE author (
  author_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE book (
  book_id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(255) NOT NULL,
  author_id INT NOT NULL,
  ISBN CHAR(13) UNIQUE,
  FOREIGN KEY (author_id) REFERENCES author(author_id)
);

CREATE TABLE member (
  member_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE
);

CREATE TABLE loan (
  loan_id INT PRIMARY KEY AUTO_INCREMENT,
  book_id INT NOT NULL,
  member_id INT NOT NULL,
  loan_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  return_date DATETIME,
  FOREIGN KEY (book_id) REFERENCES book(book_id),
  FOREIGN KEY (member_id) REFERENCES member(member_id)
);
```

### Sample Data

```sql
INSERT INTO author (name) VALUES ('J.K. Rowling');
INSERT INTO author (name) VALUES ('J.R.R. Tolkien');

INSERT INTO book (title, author_id, ISBN) VALUES 
  ('Harry Potter', 1, '9780747532699'),
  ('Lord of the Rings', 2, '9780544003415');

INSERT INTO member (name, email) VALUES 
  ('Alice', 'alice@example.com'),
  ('Bob', 'bob@example.com');

INSERT INTO loan (book_id, member_id) VALUES (1, 1), (2, 1), (1, 2);
```

### Querying Related Data

```sql
-- Find all books borrowed by Alice
SELECT b.title, b.author_id, l.loan_date
FROM loan l
JOIN book b ON l.book_id = b.book_id
JOIN member m ON l.member_id = m.member_id
WHERE m.name = 'Alice';

-- Find all books by Tolkien that are currently borrowed
SELECT b.title, m.name
FROM loan l
JOIN book b ON l.book_id = b.book_id
JOIN member m ON l.member_id = m.member_id
JOIN author a ON b.author_id = a.author_id
WHERE a.name = 'Tolkien' AND l.return_date IS NULL;
```

## Advantages of Relational Model

✅ **Data Integrity** - Relationships maintained through keys
✅ **No Redundancy** - Data stored once, referenced many times
✅ **Flexibility** - Easy to add relationships
✅ **Query Power** - JOIN queries combine multiple tables
✅ **Scalability** - Handles large datasets efficiently
✅ **Standardization** - SQL is universal

## Disadvantages

❌ **Complexity** - Relationships require careful design
❌ **Performance** - Joins can be slower than denormalized data
❌ **Overhead** - Normalization requires careful planning

## Key Takeaways

✅ Relational model = Tables with rows and columns
✅ Primary keys uniquely identify rows
✅ Foreign keys create relationships between tables
✅ Relationships prevent data redundancy
✅ Keys enforce data integrity
✅ SQL queries join tables to combine data

## Next Step

Learn about **[Database Diagrams](9-model-diagram.md)** - How to visualize and design databases.
