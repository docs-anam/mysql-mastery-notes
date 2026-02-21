# Database Applications

## What are Database Applications?

**Database Applications** are real-world systems that use databases. Understanding common patterns helps you design better databases.

## Categories of Database Applications

### 1. OLTP (Online Transaction Processing)

**Purpose**: Handle real-time business operations

**Characteristics:**
- ✅ Many short, quick transactions
- ✅ Normalized data (3NF)
- ✅ High concurrency (many users simultaneously)
- ✅ Data consistency critical (ACID)
- ✅ Moderate data volumes per transaction

**Examples:**
- E-commerce: Orders, payments, inventory
- Banking: Transfers, withdrawals, deposits
- Retail: Point of sale systems
- Reservations: Hotels, flights, restaurants

**Database Design:**
```sql
-- Normalized structure
CREATE TABLE customer (...);
CREATE TABLE orders (...);
CREATE TABLE order_item (...);
CREATE TABLE payment (...);
```

### 2. OLAP (Online Analytical Processing)

**Purpose**: Analyze business data, generate reports

**Characteristics:**
- ✅ Few complex, long-running queries
- ✅ Denormalized data (star/snowflake)
- ✅ Read-only or read-mostly
- ✅ Large data volumes
- ✅ Performance over consistency

**Examples:**
- Data warehouses: Business intelligence
- Analytics: User behavior, sales trends
- Reporting: Monthly/yearly reports
- Dashboards: Real-time metrics

**Database Design:**
```sql
-- Denormalized structure
CREATE TABLE fact_sales (
  date_id INT,
  product_id INT,
  customer_id INT,
  amount DECIMAL(10,2),
  quantity INT
);

CREATE TABLE dim_product (...);
CREATE TABLE dim_customer (...);
```

### 3. Hybrid Systems

**Purpose**: Both transactional and analytical

**Challenges:**
- Normalized for transactions
- Denormalized for analytics
- Keep them in sync

**Solution**: 
- Separate OLTP and OLAP databases
- Replicate/sync data between them
- ETL (Extract-Transform-Load) process

## Real-World Examples

### Example 1: E-commerce Platform

#### OLTP Database (Order Processing)
```
Heavily normalized, ACID compliance required

Customer → Orders → Order Items → Products
Payment → Shipping → Returns
```

#### OLAP Database (Analytics)
```
Denormalized, aggregated

Sales by region (pre-calculated)
Product performance metrics
Customer lifetime value
```

#### Sync Process
```
OLTP (Normalized)
     ↓ ETL Process
OLAP (Denormalized)
```

### Example 2: Social Media Platform

#### OLTP (User Interactions)
```
Users → Posts → Comments → Likes
Follow relationships
Messaging
```

**Design**: Normalized, but with strategic denormalization
- Post text + author name cached (denormalized)
- User follower count cached

#### OLAP (Analytics)
```
Trending posts
User engagement metrics
Growth analytics
Ad performance
```

### Example 3: Banking System

#### OLTP (Critical - Maximum Consistency)
```
Accounts table
Transactions table (immutable)
Balance = SUM(all transactions)
```

**Why**: Every dollar must be accounted for. No denormalization for consistency.

```sql
CREATE TABLE account (
  account_id INT PRIMARY KEY,
  balance DECIMAL(15, 2)  -- Could be denormalized, but risky!
);

-- Better: Calculate from transactions
SELECT SUM(amount) FROM transaction WHERE account_id = 1;
```

#### OLAP (Fraud Detection)
```
Customer spending patterns
Unusual transaction alerts
```

## Database Characteristics by Application

| Aspect | OLTP | OLAP |
|--------|------|------|
| Data structure | Normalized | Denormalized |
| Transaction size | Small | Large |
| Response time | < 1 second | Seconds/minutes |
| Users | Many (100s-1000s) | Few (10s-100s) |
| Queries | Simple | Complex |
| Updates | Frequent | Batch/Scheduled |
| Consistency | Critical | Approximate OK |
| Typical DB | MySQL, PostgreSQL | Data Warehouse, Hadoop |

## Common Application Patterns

### Pattern 1: E-commerce

**Entities:**
- Customer: Users of the platform
- Product: Items for sale
- Order: Purchase record
- OrderItem: Items in an order
- Payment: Payment method and history
- Review: Customer reviews

**Key Relationships:**
```
Customer ──[1:M]── Order ──[1:M]── OrderItem ──[M:1]── Product
         ──[1:M]── Payment
Product ──[1:M]── Review ──[M:1]── Customer
```

**Critical Queries:**
- Find products by category
- Get order history for customer
- Update inventory
- Process payment

### Pattern 2: Social Media

**Entities:**
- User: Account and profile
- Post: Content created
- Comment: Response to post
- Like: Vote on post/comment
- Follow: User subscription

**Key Relationships:**
```
User ──[1:M]── Post ──[1:M]── Comment
    ──[1:M]── Like (on Post)
    ──[M:N]── Follow
```

**Critical Queries:**
- Get feed (posts from followed users)
- Post comment
- Get likes count
- Find users

### Pattern 3: Content Management System

**Entities:**
- User: Authors and editors
- Page: Published content
- Category: Organization
- Tag: Labels
- Comment: Page comments
- Version: Edit history

**Key Relationships:**
```
User ──[1:M]── Page ──[1:M]── Version
     ──[1:M]── Comment
Page ──[M:N]── Category
    ──[M:N]── Tag
```

### Pattern 4: Inventory Management

**Entities:**
- Product: Items
- Warehouse: Storage locations
- Inventory: Stock levels
- Supplier: Vendors
- PurchaseOrder: Resupply
- Movement: Stock transfers

**Key Relationships:**
```
Product ──[1:M]── Inventory ──[M:1]── Warehouse
        ──[M:N]── Supplier
Supplier ──[1:M]── PurchaseOrder ──[1:M]── PurchaseOrderItem
```

**Critical Queries:**
- Get stock level
- Find low-stock products
- Transfer between warehouses
- Purchase order management

## Best Practices by Application Type

### For OLTP Systems

✅ **Normalize to 3NF** - Data integrity first
✅ **Use transactions** - Ensure consistency
✅ **Index frequently queried columns** - Speed up reads
✅ **Use appropriate locks** - Control concurrency
✅ **Plan for growth** - Scalability
✅ **Backup regularly** - Data safety

### For OLAP Systems

✅ **Denormalize strategically** - Performance first
✅ **Pre-aggregate data** - Summary tables
✅ **Use compression** - Save storage
✅ **Partition by date** - Manage large tables
✅ **Use columnar storage** - Better for analytics
✅ **Batch load data** - Efficient ETL

### For Hybrid Systems

✅ **Separate databases** - Different designs
✅ **Replicate from OLTP to OLAP** - Keep in sync
✅ **ETL process** - Transform as needed
✅ **Cache critical data** - Performance boost
✅ **Monitor both** - Ensure consistency

## Application-Database Interaction

### OLTP Application Code Pattern

```python
# Process order
try:
    db.begin_transaction()
    
    # Check inventory
    product = db.query("SELECT * FROM product WHERE id = ?", product_id)
    if product.stock < quantity:
        raise InsufficientStock()
    
    # Create order
    order = db.insert("orders", {
        'customer_id': customer_id,
        'total': quantity * product.price
    })
    
    # Create order items
    db.insert("order_item", {
        'order_id': order.id,
        'product_id': product_id,
        'quantity': quantity,
        'unit_price': product.price
    })
    
    # Update inventory
    db.execute(
        "UPDATE product SET stock = stock - ? WHERE id = ?",
        quantity, product_id
    )
    
    # Create payment record
    db.insert("payment", {
        'order_id': order.id,
        'amount': order.total
    })
    
    db.commit()
    
except Exception as e:
    db.rollback()
    raise
```

### OLAP Query Pattern

```sql
-- Analyze sales trends
SELECT 
  DATE_TRUNC(order_date, MONTH) as month,
  category,
  SUM(total_amount) as revenue,
  COUNT(DISTINCT customer_id) as customers,
  AVG(total_amount) as avg_order_value
FROM fact_sales
JOIN dim_product ON fact_sales.product_id = dim_product.product_id
WHERE year(order_date) = 2024
GROUP BY month, category
ORDER BY month DESC, revenue DESC;
```

## Choosing the Right Design

### Ask These Questions:

1. **Read vs Write Ratio**
   - More reads? → Consider denormalization
   - More writes? → Stay normalized

2. **Data Change Frequency**
   - Static data? → Can denormalize
   - Frequently updated? → Keep normalized

3. **Consistency Requirements**
   - Critical (banking)? → Strict normalization
   - Approximate OK (analytics)? → Denormalize

4. **Performance Requirements**
   - Sub-second response? → Denormalize aggressively
   - Seconds acceptable? → Standard normalization

5. **Scale**
   - Millions of rows? → Need careful design
   - Thousands? → Standard design fine

## Key Takeaways

✅ OLTP = Transactional, normalized, consistent
✅ OLAP = Analytical, denormalized, performance-focused
✅ Real applications often need both
✅ Understand your data access patterns
✅ Design for your application's primary use case
✅ Keep OLTP and OLAP separate when needed

## Summary of Section

You've now learned:
1. **[Introduction](1-intro.md)** - Overview
2. **[What is Database](2-database.md)** - Fundamentals
3. **[Database Systems](3-database-system.md)** - How they work
4. **[Data Modeling](4-data-modeling.md)** - Design process
5. **[Attributes & Types](5-attributes-type.md)** - Data types
6. **[Keys & Constraints](6-attribute-key.md)** - Relationships
7. **[Entity Relationship Diagrams](7-erd.md)** - Visualization
8. **[Relational Model](8-model-data-relational.md)** - Tables & relations
9. **[Database Diagrams](9-model-diagram.md)** - Complete diagrams
10. **[Normalization](10-data-normalization.md)** - Principles
11. **[First Normal Form (1NF)](11-normal-form-1.md)** - Atomic data
12. **[Second Normal Form (2NF)](12-normal-form-2.md)** - No partial dependencies
13. **[Third Normal Form (3NF)](13-normal-form-3.md)** - No transitive dependencies
14. **[Denormalization](14-denormalization.md)** - Performance optimization
15. **[Database Applications](15-database-applications.md)** - Real-world usage

You now have the foundation to design professional databases!
