# Data Modeling

## What is Data Modeling?

**Data Modeling** is the process of designing the structure of your database BEFORE implementation. It's the blueprint for how your data will be organized.

## Why Data Modeling Matters

### Without Proper Modeling
- ❌ Duplicate data everywhere
- ❌ Inconsistent information
- ❌ Difficult to query and analyze
- ❌ Performance problems
- ❌ Hard to maintain and extend
- ❌ Data anomalies and corruption

### With Good Modeling
- ✅ Clean, organized structure
- ✅ Single source of truth
- ✅ Fast queries and good performance
- ✅ Easy to maintain and extend
- ✅ Consistent data
- ✅ Scalable design

## Three Levels of Data Modeling

### 1. Conceptual Model
- **What**: High-level overview
- **Focus**: Business concepts and relationships
- **Who**: Business analysts, stakeholders
- **Tools**: Mind maps, simple diagrams
- **Example**: "We have Customers and Orders"

### 2. Logical Model
- **What**: Detailed structure without implementation details
- **Focus**: Entities, attributes, relationships, constraints
- **Who**: Database designers, architects
- **Tools**: Entity-Relationship Diagrams (ERD)
- **Example**: "Customers table has ID, Name, Email; Orders table has ID, CustomerID, Date"

### 3. Physical Model
- **What**: Implementation-specific design
- **Focus**: Actual tables, columns, data types, indexes, performance
- **Who**: Database administrators, developers
- **Tools**: DDL (CREATE TABLE statements)
- **Example**: Actual MySQL CREATE TABLE with data types, constraints

## The Modeling Process

```
1. Understand Requirements
   ↓
2. Identify Entities
   ↓
3. Define Attributes
   ↓
4. Establish Relationships
   ↓
5. Create ERD (Visual)
   ↓
6. Normalize (Remove redundancy)
   ↓
7. Create Physical Schema
   ↓
8. Implement & Test
```

## Key Concepts

### Entity
- **What**: A thing/object you want to store data about
- **Example**: Customer, Product, Order, Employee
- Becomes a **table** in database

### Attribute
- **What**: Property or characteristic of an entity
- **Example**: Customer has Name, Email, Phone
- Becomes a **column** in database

### Relationship
- **What**: How entities are connected
- **Types**: One-to-One, One-to-Many, Many-to-Many
- Becomes **foreign keys** in database

## Example: Simple E-commerce Model

### Conceptual Level
- We have Customers
- We have Products
- Customers place Orders
- Orders contain Products

### Logical Level
```
CUSTOMER
├── CustomerID (unique identifier)
├── Name
├── Email
└── Phone

PRODUCT
├── ProductID
├── Name
├── Price
└── Stock

ORDER
├── OrderID
├── CustomerID (references CUSTOMER)
├── OrderDate
└── TotalAmount

ORDER_ITEM
├── OrderItemID
├── OrderID (references ORDER)
├── ProductID (references PRODUCT)
└── Quantity
```

### Physical Level
```sql
CREATE TABLE customer (
  customer_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE,
  phone VARCHAR(15)
);

CREATE TABLE product (
  product_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  price DECIMAL(10, 2),
  stock INT DEFAULT 0
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
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (product_id) REFERENCES product(product_id)
);
```

## Best Practices

✅ **Start simple** - Don't over-complicate initially
✅ **Understand requirements** - Talk to stakeholders
✅ **Think about relationships** - How do entities connect?
✅ **Consider queries** - How will data be accessed?
✅ **Plan for growth** - Design for future needs
✅ **Document decisions** - Explain your choices
✅ **Get feedback** - Review with team

## Common Modeling Mistakes

❌ **No primary keys** - Always have unique identifiers
❌ **Duplicate data** - Store once, reference many times
❌ **Wrong relationships** - Misunderstanding how data connects
❌ **Poor naming** - Use clear, consistent names
❌ **Missing constraints** - Not enforcing data rules
❌ **Ignoring queries** - Design without considering how data is used

## Key Takeaways

✅ Data modeling = Blueprint before implementation
✅ Three levels: Conceptual, Logical, Physical
✅ Entities → Tables, Attributes → Columns, Relationships → Keys
✅ Good modeling = Easy to query, maintain, scale

## Next Steps

1. **[Attributes and Data Types](5-attributes-type.md)** - Learn about data types
2. **[Keys and Constraints](6-attribute-key.md)** - Understand relationship keys
3. **[Entity Relationship Diagrams](7-erd.md)** - Visual representation
