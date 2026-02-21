# Keys and Constraints

## What Are Keys?

**Keys** are attributes (or combinations of attributes) that uniquely identify records and establish relationships between tables.

## Primary Key

### Definition
A **Primary Key** is an attribute that **uniquely identifies** each record in a table. Every table should have exactly one primary key.

### Characteristics
- ✅ **Unique**: No two rows can have the same primary key
- ✅ **Non-NULL**: Every row must have a value
- ✅ **Immutable**: Value shouldn't change
- ✅ **Single per table**: Only one primary key per table

### Example: Customer Table

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,  -- Primary Key
  name VARCHAR(100),
  email VARCHAR(100)
);

-- Auto-increment creates: 1, 2, 3, 4...
INSERT INTO customer (name, email) VALUES ('John', 'john@example.com');  -- ID=1
INSERT INTO customer (name, email) VALUES ('Jane', 'jane@example.com');  -- ID=2
```

### Types of Primary Keys

#### Natural Key
- Based on existing business data
- Example: Email address (usually unique)

```sql
CREATE TABLE user (
  email VARCHAR(100) PRIMARY KEY,
  name VARCHAR(100),
  phone VARCHAR(15)
);
```

#### Surrogate Key
- Artificial, auto-generated key
- No business meaning
- Most common and recommended

```sql
CREATE TABLE user (
  user_id INT PRIMARY KEY AUTO_INCREMENT,  -- Surrogate
  email VARCHAR(100) UNIQUE,               -- Natural but unique
  name VARCHAR(100)
);
```

### Primary Key vs Unique Constraint

| Feature | Primary Key | Unique |
|---------|-------------|--------|
| Uniqueness | Yes | Yes |
| NULL allowed | NO | YES (multiple NULLs) |
| Per table | Only 1 | Multiple allowed |
| Index created | Automatic | Automatic |
| Performance | Fastest | Fast |
| Use for | Identify rows | Prevent duplicates |

## Foreign Key

### Definition
A **Foreign Key** is an attribute that references the primary key of another table. It establishes relationships between tables.

### Characteristics
- ✅ Points to primary key of another table
- ✅ Enforces referential integrity
- ✅ Prevents invalid data
- ✅ Can be NULL (optional relationship)
- ✅ Multiple foreign keys per table allowed

### Example: Customer and Orders

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100),
  email VARCHAR(100)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
  -- Foreign key: orders.customer_id → customer.customer_id
);
```

### What Foreign Keys Prevent

```sql
-- Without Foreign Key (BAD):
INSERT INTO orders (customer_id, order_date) VALUES (999, NOW());
-- ❌ CustomerID 999 doesn't exist! Orphan order!

-- With Foreign Key (SAFE):
INSERT INTO orders (customer_id, order_date) VALUES (999, NOW());
-- ✅ ERROR! Customer 999 doesn't exist. Rejected!
```

### Referential Integrity Actions

```sql
CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  order_date TIMESTAMP,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
    ON DELETE CASCADE         -- Delete orders if customer deleted
    ON UPDATE CASCADE         -- Update customer_id if changed
);
```

**Options:**
- `CASCADE`: Delete/update foreign table records too
- `RESTRICT`: Prevent deletion/update if referenced
- `SET NULL`: Set foreign key to NULL
- `NO ACTION`: Default, same as RESTRICT

## Unique Constraint

### Definition
A **Unique Constraint** ensures that all values in an attribute are unique (no duplicates).

### Differences from Primary Key
- Allows multiple NULL values
- Multiple unique constraints per table
- Slightly less efficient than primary key

### Example: Email Uniqueness

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(100) UNIQUE NOT NULL,  -- Must be unique
  name VARCHAR(100)
);

-- This is allowed:
INSERT INTO customer (email, name) VALUES ('john@example.com', 'John');
INSERT INTO customer (email, name) VALUES ('jane@example.com', 'Jane');

-- This fails:
INSERT INTO customer (email, name) VALUES ('john@example.com', 'Another John');
-- ❌ ERROR! Duplicate email!
```

## Composite Key

### Definition
A **Composite Key** (or compound key) is a primary key made up of multiple attributes.

### When to Use
- Represents multi-dimensional uniqueness
- Models junction tables
- Business rules require it

### Example: Order Items

```sql
CREATE TABLE order_item (
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  quantity INT,
  price DECIMAL(10, 2),
  PRIMARY KEY (order_id, product_id)  -- Composite key
);

-- Same order can't have duplicate products:
INSERT INTO order_item VALUES (1, 101, 5, 29.99);  -- ✅ OK
INSERT INTO order_item VALUES (1, 101, 3, 29.99);  -- ❌ Duplicate (order 1, product 101)
```

## Other Constraints

### NOT NULL
Ensures a field always has a value.

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,  -- Must have value
  phone VARCHAR(15)            -- Can be NULL
);
```

### DEFAULT
Provides a default value if not specified.

```sql
CREATE TABLE order (
  order_id INT PRIMARY KEY,
  status ENUM('pending', 'completed') DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE
);
```

### CHECK
Validates data against a condition.

```sql
CREATE TABLE product (
  product_id INT PRIMARY KEY,
  name VARCHAR(100),
  price DECIMAL(10, 2),
  CHECK (price > 0)  -- Price must be positive
);
```

### AUTO_INCREMENT
Automatically generates unique values.

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,  -- 1, 2, 3, ...
  name VARCHAR(100)
);
```

## Key Relationships

### One-to-Many
One customer has many orders.

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);

-- One customer_id can appear in many orders rows
```

### Many-to-Many
A student takes many courses, a course has many students.

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
  enrollment_date DATE,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES student(student_id),
  FOREIGN KEY (course_id) REFERENCES course(course_id)
);
```

### One-to-One
A person has one passport, a passport belongs to one person.

```sql
CREATE TABLE person (
  person_id INT PRIMARY KEY
);

CREATE TABLE passport (
  passport_id INT PRIMARY KEY,
  person_id INT UNIQUE NOT NULL,  -- UNIQUE for one-to-one
  FOREIGN KEY (person_id) REFERENCES person(person_id)
);
```

## Best Practices

✅ **Always have a primary key** - Every table needs one
✅ **Use surrogate keys** - Auto-increment IDs are usually best
✅ **Use foreign keys** - Maintain referential integrity
✅ **Name consistently** - customer_id, order_id, etc.
✅ **Cascade or restrict** - Decide on deletion behavior
✅ **Use UNIQUE for business uniqueness** - Email, username, etc.
✅ **Document relationships** - Make it clear which tables relate

## Common Mistakes

❌ **No primary key** - Every table needs one
❌ **Multiple primary keys** - Only one per table
❌ **Missing foreign keys** - Allows data inconsistency
❌ **Bad cascade rules** - Can accidentally delete too much
❌ **Wrong relationship type** - One-to-many vs many-to-many confusion

## Example: Complete Schema

```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(100) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  phone VARCHAR(15)
);

CREATE TABLE product (
  product_id INT PRIMARY KEY AUTO_INCREMENT,
  sku CHAR(10) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL CHECK (price > 0)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id) ON DELETE CASCADE
);

CREATE TABLE order_item (
  order_item_id INT PRIMARY KEY AUTO_INCREMENT,
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  quantity INT NOT NULL CHECK (quantity > 0),
  FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
  FOREIGN KEY (product_id) REFERENCES product(product_id)
);
```

## Key Takeaways

✅ Primary Key = Unique identifier for each row
✅ Foreign Key = Reference to another table's primary key
✅ Unique Constraint = No duplicates allowed
✅ Composite Key = Multiple attributes make it unique
✅ Foreign keys maintain referential integrity

## Next Step

Learn about **[Entity Relationship Diagrams (ERD)](7-erd.md)** - How to visualize these relationships.
