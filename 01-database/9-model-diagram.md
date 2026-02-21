# Database Diagrams

## What are Database Diagrams?

**Database Diagrams** are visual representations of database structure showing tables, columns, data types, and relationships. They're essential for:

- ✅ Understanding database structure at a glance
- ✅ Communicating design to team members
- ✅ Planning modifications
- ✅ Identifying design issues
- ✅ Documentation

## Types of Database Diagrams

### 1. Entity-Relationship Diagram (ERD)

Shows entities and their relationships.

```
CUSTOMER ──[1:M]── ORDER ──[M:1]── PRODUCT
```

**Best for:** High-level overview, design planning

### 2. Schema Diagram

Detailed diagram with tables, columns, and data types.

```
┌──────────────────────┐
│      CUSTOMER        │
├──────────────────────┤
│ customer_id : INT    │◄─────┐
│ name : VARCHAR       │      │ FK
│ email : VARCHAR      │      │
└──────────────────────┘      │
                              │
                         ┌────┴──────────────┐
                         │      ORDERS       │
                         ├───────────────────┤
                         │ order_id : INT    │
                         │ customer_id : INT │
                         │ order_date : DATE │
                         └───────────────────┘
```

**Best for:** Implementation details, SQL design

### 3. Crow's Foot Diagram

Shows cardinality with special notation.

```
CUSTOMER ─○─ ORDERS
(zero or one)

CUSTOMER ─|─ ORDERS
(exactly one)

CUSTOMER ─◇─ ORDERS
(zero or many)

CUSTOMER ─<─ ORDERS
(one or many)
```

**Best for:** Precise relationship definitions

## Creating Database Diagrams

### Step 1: Define Tables

```
Tables:
- CUSTOMER
- PRODUCT
- ORDER
- ORDER_ITEM
```

### Step 2: Add Columns & Data Types

```
CUSTOMER
├── customer_id (INT PRIMARY KEY)
├── name (VARCHAR(100))
├── email (VARCHAR(100))
└── phone (VARCHAR(15))
```

### Step 3: Identify Keys

```
CUSTOMER
├── customer_id (INT PRIMARY KEY) ← Primary Key
├── email (UNIQUE) ← Unique Key
├── name (VARCHAR(100))
└── phone (VARCHAR(15))
```

### Step 4: Add Relationships

```
CUSTOMER → ORDER (customer_id)
PRODUCT → ORDER_ITEM (product_id)
ORDER → ORDER_ITEM (order_id)
```

### Step 5: Draw Diagram

```
┌─────────────────────────┐          ┌──────────────────────┐
│      CUSTOMER           │          │      ORDERS          │
├─────────────────────────┤          ├──────────────────────┤
│ customer_id (PK)        │◄────[FK]─┤ order_id (PK)        │
│ name                    │          │ customer_id (FK)     │
│ email (UNIQUE)          │          │ order_date           │
│ phone                   │          │ total_amount         │
└─────────────────────────┘          └──────────────────────┘
                                             │
                                        [PK] │ [FK]
                                             │
                                 ┌───────────▼──────────────┐
                                 │     ORDER_ITEM           │
                                 ├────────────────────────┐ │
                                 │ order_item_id (PK)     │ │
                                 │ order_id (FK)          │ │
                                 │ product_id (FK)        │ │
                                 │ quantity               │ │
                                 │ unit_price             │ │
                                 └────────────────────────┴─┘
                                             │
                                        [FK] │
                                             │
                                 ┌───────────▼──────────────┐
                                 │      PRODUCT            │
                                 ├────────────────────────┐ │
                                 │ product_id (PK)        │ │
                                 │ name                   │ │
                                 │ price                  │ │
                                 │ stock                  │ │
                                 │ category               │ │
                                 └────────────────────────┴─┘
```

## Detailed Schema Diagram

### With All Details

```
╔═══════════════════════════════════╗
║         CUSTOMER TABLE            ║
╠════════════════════╦═══════════════╣
║ Column Name        ║ Data Type     ║
╟────────────────────╫───────────────╢
║ customer_id (PK)   ║ INT           ║
║ name               ║ VARCHAR(100)  ║
║ email (UNIQUE)     ║ VARCHAR(100)  ║
║ phone              ║ VARCHAR(15)   ║
║ created_at         ║ TIMESTAMP     ║
║ is_active          ║ BOOLEAN       ║
╚═══════════════════════════════════╝
         ▲
         │ FK: customer_id
         │
╔═══════════════════════════════════╗
║          ORDERS TABLE             ║
╠════════════════════╦═══════════════╣
║ Column Name        ║ Data Type     ║
╟────────────────────╫───────────────╢
║ order_id (PK)      ║ INT           ║
║ customer_id (FK)   ║ INT           ║
║ order_date         ║ TIMESTAMP     ║
║ total_amount       ║ DECIMAL(10,2) ║
║ status             ║ ENUM          ║
╚═══════════════════════════════════╝
```

## Database Diagram Tools

### Online Tools (Free)
- **Draw.io** - Free, simple, database templates
- **Lucidchart** - Free tier available, professional
- **Miro** - Whiteboard collaboration
- **DBDiagram.io** - SQL-focused, simple syntax

### Professional Tools
- **MySQL Workbench** - Free, MySQL-specific
- **DBeaver** - Free, all databases
- **DataGrip** - Paid, powerful JetBrains tool
- **Navicat** - Paid, visual database designer

### Design-Only Tools
- **Figma** - Design tool with database templates
- **Visio** - Microsoft tool with database shapes

## Example: Complete E-commerce Database Diagram

### Text Format (ASCII Art)

```
┏━━━━━━━━━━━━━━━━━━━━━┓
┃     CATEGORY        ┃
┣━━━━━━━━━━━━━━━━━━━━━┫
┃ category_id (PK)    ┃
┃ name                ┃
┃ description         ┃
┗━━━━━━━━━━━━━━━━━━━━━┛
         ▲
         │[1:M]
         │
┏━━━━━━━━━━━━━━━━━━━━━┓          ┏━━━━━━━━━━━━━━━━━━━━━┓
┃      PRODUCT        ┃◄────[FK]─┤     INVENTORY       ┃
┣━━━━━━━━━━━━━━━━━━━━━┫          ┣━━━━━━━━━━━━━━━━━━━━━┫
┃ product_id (PK)     ┃          ┃ inventory_id (PK)   ┃
┃ category_id (FK)    ┃          ┃ product_id (FK)     ┃
┃ name                ┃          ┃ warehouse           ┃
┃ price               ┃          ┃ quantity            ┃
┃ description         ┃          ┗━━━━━━━━━━━━━━━━━━━━━┛
┗━━━━━━━━━━━━━━━━━━━━━┛
         ▲
         │[1:M]
         │
┏━━━━━━━━━━━━━━━━━━━━━┓          ┏━━━━━━━━━━━━━━━━━━━━━┓
┃      CUSTOMER       ┃          ┃        ORDERS       ┃
┣━━━━━━━━━━━━━━━━━━━━━┫          ┣━━━━━━━━━━━━━━━━━━━━━┫
┃ customer_id (PK)    ┃◄────[FK]─┤ order_id (PK)       ┃
┃ name                ┃          ┃ customer_id (FK)    ┃
┃ email               ┃          ┃ order_date          ┃
┃ phone               ┃          ┃ total_amount        ┃
┗━━━━━━━━━━━━━━━━━━━━━┛          ┗━━━━━━━━━━━━━━━━━━━━━┛
                                         ▲
                                    [1:M]│
                                         │
                        ┏━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━┓
                        ┃     ORDER_ITEM              ┃
                        ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
                        ┃ order_item_id (PK)          ┃
                        ┃ order_id (FK)               ┃
                        ┃ product_id (FK)─────────┐   ┃
                        ┃ quantity                 │   ┃
                        ┃ unit_price               │   ┃
                        ┗━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━┛
                                               [M:1]
```

## SQL DDL from Diagram

Once diagram is created, generate SQL:

```sql
CREATE TABLE category (
  category_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  description TEXT
);

CREATE TABLE product (
  product_id INT PRIMARY KEY AUTO_INCREMENT,
  category_id INT NOT NULL,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  description TEXT,
  FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE,
  phone VARCHAR(15)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY AUTO_INCREMENT,
  customer_id INT NOT NULL,
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  total_amount DECIMAL(10, 2),
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);

CREATE TABLE order_item (
  order_item_id INT PRIMARY KEY AUTO_INCREMENT,
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  unit_price DECIMAL(10, 2) NOT NULL,
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (product_id) REFERENCES product(product_id)
);

CREATE TABLE inventory (
  inventory_id INT PRIMARY KEY AUTO_INCREMENT,
  product_id INT NOT NULL,
  warehouse VARCHAR(100),
  quantity INT DEFAULT 0,
  FOREIGN KEY (product_id) REFERENCES product(product_id)
);
```

## Best Practices

✅ **Show all columns** - Include every column
✅ **Mark keys** - Clearly show primary and foreign keys
✅ **Label relationships** - Show cardinality
✅ **Use data types** - Include column data types
✅ **Organize layout** - Related tables near each other
✅ **Keep current** - Update as schema changes
✅ **Document constraints** - Show business rules

## Common Mistakes

❌ **Too simplified** - Missing important details
❌ **Too complicated** - So detailed it's unreadable
❌ **Inconsistent notation** - Mixing multiple styles
❌ **No keys shown** - Can't understand relationships
❌ **Wrong cardinality** - Relationships misrepresented
❌ **Missing constraints** - Not showing NOT NULL, UNIQUE, etc.

## Key Takeaways

✅ Diagrams visualize database structure
✅ Include tables, columns, keys, and relationships
✅ Use consistent notation and tools
✅ Keep diagrams updated with schema
✅ Use diagrams for communication and planning

## Next Step

Learn about **[Normalization](10-data-normalization.md)** - Organizing data efficiently.
