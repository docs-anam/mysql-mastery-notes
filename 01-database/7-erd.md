# Entity Relationship Diagrams (ERD)

## What is an ERD?

An **Entity Relationship Diagram (ERD)** is a visual representation of entities and their relationships in a database.

### Benefits
- ✅ Clear visualization of data structure
- ✅ Easy communication with non-technical stakeholders
- ✅ Identify design issues before implementation
- ✅ Documentation of database design
- ✅ Plan for future modifications

## ERD Components

### 1. Entity (Table)

Represented as a **rectangle** with the entity name.

```
┌─────────────────┐
│    CUSTOMER     │
├─────────────────┤
│ customer_id (PK)│
│ name            │
│ email           │
│ phone           │
└─────────────────┘
```

### 2. Attribute (Column)

Listed inside the entity box.

```
CUSTOMER
├── customer_id ← PRIMARY KEY (underlined or marked as PK)
├── name
├── email ← UNIQUE (may be marked)
└── phone
```

### 3. Relationship (Connection Line)

Lines connecting entities show how they relate.

```
CUSTOMER ─────── ORDERS
```

### 4. Relationship Types (Cardinality)

Shows **how many** of each entity relates to the other.

| Notation | Meaning | Example |
|----------|---------|---------|
| 1:1 | One-to-One | One person, one passport |
| 1:N (1:M) | One-to-Many | One customer, many orders |
| M:N | Many-to-Many | Many students, many courses |

## Cardinality Notation

### Chen's Notation (Classic)
```
1 ── (relationship) ── M  (One to Many)
```

### Crow's Foot Notation (Popular)
```
─○ (zero or one)
─| (exactly one)
─◇ (zero or many)
─< (one or many)
```

## Example: E-commerce Database

### Full ERD with Crow's Foot

```
┌──────────────────┐              ┌──────────────────┐
│    CUSTOMER      │              │    ORDERS        │
├──────────────────┤              ├──────────────────┤
│ customer_id (PK) ├─────[1]──[M]─┤ order_id (PK)    │
│ name             │              │ customer_id (FK) │
│ email (UNIQUE)   │              │ order_date       │
│ phone            │              │ total_amount     │
└──────────────────┘              └──────────────────┘
                                         │
                                         │ [1]──[M]
                                         │
                                    ┌────▼──────────────┐
                                    │   ORDER_ITEM      │
                                    ├───────────────────┤
                                    │ order_item_id (PK)│
                                    │ order_id (FK)     │
                                    │ product_id (FK)   │
                                    │ quantity          │
                                    └───────────────────┘
                                         │
                                         │ [1]──[M]
                                         │
                                    ┌────▼──────────────┐
                                    │     PRODUCT       │
                                    ├───────────────────┤
                                    │ product_id (PK)   │
                                    │ name              │
                                    │ price             │
                                    │ stock             │
                                    └───────────────────┘
```

## Reading the Relationships

### CUSTOMER to ORDERS
- **Reading left-to-right**: "One CUSTOMER places many ORDERS"
- **Reading right-to-left**: "Many ORDERS belong to one CUSTOMER"
- **Type**: One-to-Many (1:M)

### ORDERS to ORDER_ITEM
- **Reading left-to-right**: "One ORDER contains many ORDER_ITEMs"
- **Reading right-to-left**: "Many ORDER_ITEMs belong to one ORDER"
- **Type**: One-to-Many (1:M)

### PRODUCT to ORDER_ITEM
- **Reading left-to-right**: "One PRODUCT appears in many ORDER_ITEMs"
- **Reading right-to-left**: "Many ORDER_ITEMs reference one PRODUCT"
- **Type**: One-to-Many (1:M)

## Types of Relationships

### One-to-One (1:1)

```
┌──────────────┐         ┌──────────────┐
│   PERSON     │         │   PASSPORT   │
├──────────────┤         ├──────────────┤
│ person_id(PK)├─[1]──[1]─┤ passport_id  │
│ name         │         │ person_id(FK)│
└──────────────┘         └──────────────┘

Usage: One person has one passport, 
       one passport belongs to one person
```

### One-to-Many (1:N or 1:M)

```
┌──────────────┐         ┌──────────────┐
│ DEPARTMENT   │         │   EMPLOYEE   │
├──────────────┤         ├──────────────┤
│ dept_id(PK)  ├─[1]──[M]─┤ emp_id(PK)   │
│ dept_name    │         │ dept_id(FK)  │
└──────────────┘         │ name         │
                         └──────────────┘

Usage: One department has many employees,
       one employee belongs to one department
```

### Many-to-Many (M:N)

```
┌──────────────┐         ┌────────────────┐         ┌──────────────┐
│   STUDENT    │         │   ENROLLMENT   │         │    COURSE    │
├──────────────┤         ├────────────────┤         ├──────────────┤
│ student_id(PK)├─[M]──[1]─┤ student_id(FK)│         │ course_id(PK)│
│ name         │         │ course_id(FK)  ├─[1]──[M]─┤ course_name  │
└──────────────┘         │ enrollment_date│         └──────────────┘
                         └────────────────┘

Usage: Many students take many courses,
       many courses have many students
       (Requires junction/bridge table: ENROLLMENT)
```

## Creating ERD in Practice

### Step 1: Identify Entities

```
Entities: Customer, Product, Order, OrderItem
```

### Step 2: Identify Attributes

```
Customer: customer_id, name, email, phone
Product: product_id, name, price, stock
Order: order_id, customer_id, order_date
OrderItem: order_item_id, order_id, product_id, quantity
```

### Step 3: Identify Primary Keys

```
Customer.customer_id (PK)
Product.product_id (PK)
Order.order_id (PK)
OrderItem.order_item_id (PK)
```

### Step 4: Identify Foreign Keys & Relationships

```
Order.customer_id (FK) → Customer.customer_id (One Customer to Many Orders)
OrderItem.order_id (FK) → Order.order_id (One Order to Many OrderItems)
OrderItem.product_id (FK) → Product.product_id (One Product to Many OrderItems)
```

### Step 5: Draw the ERD

```
CUSTOMER ──[1:M]── ORDER ──[1:M]── ORDER_ITEM ──[M:1]── PRODUCT
```

## ERD Tools

### Free Online Tools
- **Lucidchart** - Web-based, collaborative
- **Draw.io** - Free, simple, supports databases
- **Miro** - Whiteboard-style
- **ERDPlus** - Database-specific
- **Creately** - Collaborative

### SQL Tools
- **MySQL Workbench** - Free, MySQL-focused
- **DBeaver** - Free, all databases
- **SQLyog** - MySQL-focused

## Best Practices

✅ **Show primary keys** - Mark them clearly (PK)
✅ **Show foreign keys** - Mark them clearly (FK)
✅ **Label relationships** - Show cardinality
✅ **Use consistent notation** - Stick to one style
✅ **Include data types** - Optional but helpful
✅ **Keep it readable** - Not too cramped
✅ **Update with changes** - Keep ERD current

## Common Mistakes

❌ **Missing relationships** - Forgot to connect related entities
❌ **Wrong cardinality** - One-to-many shown as one-to-one
❌ **No primary keys** - Entities without identifiers
❌ **Unclear notation** - Mix of different styles
❌ **Too detailed** - So many attributes it's unreadable
❌ **Out of date** - Old ERD that doesn't match database

## From ERD to SQL

### ERD
```
CUSTOMER ──[1:M]── ORDER
```

### SQL (Implementation)
```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);
```

## Key Takeaways

✅ ERD visualizes database structure and relationships
✅ Entities become tables, attributes become columns
✅ Primary keys identify rows uniquely
✅ Foreign keys create relationships between tables
✅ Cardinality shows one-to-one, one-to-many, many-to-many
✅ ERD helps communicate design before implementation

## Next Step

Learn about **[Relational Model](8-model-data-relational.md)** - How relationships work in practice.
