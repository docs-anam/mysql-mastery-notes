# Denormalization

## What is Denormalization?

**Denormalization** is the process of intentionally adding redundant data back into a database that was previously normalized.

**Key Point**: We **violate** 1NF, 2NF, or 3NF rules **on purpose** to improve performance.

## Why Denormalize?

### The Normalization Trade-off

**Normalized Database**
- ✅ Less storage space
- ✅ No data redundancy
- ✅ Consistent data
- ❌ More JOINs needed
- ❌ More CPU usage
- ❌ Slower queries

**Denormalized Database**
- ✅ Faster queries (fewer JOINs)
- ✅ Better read performance
- ❌ More storage space
- ❌ Data redundancy
- ❌ Harder to maintain
- ❌ Risk of inconsistency

## When to Denormalize

### Denormalize When:
✅ **Read-heavy system** (much more reading than writing)
✅ **Performance critical** (e.g., real-time dashboards)
✅ **Reporting system** (data warehouse, analytics)
✅ **Cache layer** (Redis, Memcached)
✅ **Historical data** (audit logs, time-series)
✅ **Data rarely changes** (reference data)

### DON'T Denormalize When:
❌ **Write-heavy system** (frequent updates)
❌ **Consistency critical** (financial systems)
❌ **Data changes frequently** (inventory, pricing)
❌ **Storage is expensive** (limited space)

## Denormalization Techniques

### 1. Duplicate Columns

Store the same data in multiple tables to avoid JOINs.

#### Example: Customer Name in Orders

**Normalized (3NF)**
```sql
SELECT o.order_id, c.name, o.order_date
FROM orders o
JOIN customer c ON o.customer_id = c.customer_id;
-- Requires JOIN
```

**Denormalized**
```sql
CREATE TABLE orders_denorm (
  order_id INT PRIMARY KEY,
  customer_id INT,
  customer_name VARCHAR(100),  -- Duplicate from customer table
  order_date TIMESTAMP
);

-- Now single table query:
SELECT order_id, customer_name, order_date FROM orders_denorm;
-- No JOIN needed!
```

**Trade-off:**
- ✅ Faster query (no JOIN)
- ❌ Larger table
- ❌ If customer name changes, must update orders table too

### 2. Derived Columns

Pre-calculate values to avoid aggregation.

#### Example: Total Amount in Order Header

**Normalized**
```sql
SELECT o.order_id, SUM(oi.quantity * oi.unit_price) as total
FROM orders o
JOIN order_item oi ON o.order_id = oi.order_id
GROUP BY o.order_id;
-- Requires calculation on each query
```

**Denormalized**
```sql
CREATE TABLE orders_denorm (
  order_id INT PRIMARY KEY,
  customer_id INT,
  total_amount DECIMAL(10, 2),  -- Pre-calculated total
  order_date TIMESTAMP
);

-- Now instant query:
SELECT order_id, total_amount, order_date FROM orders_denorm;
-- No calculation needed!
```

**Trade-off:**
- ✅ Much faster
- ❌ Storage space
- ❌ Must keep total_amount in sync with order_items

### 3. Summarized Data

Pre-aggregate frequently needed summaries.

#### Example: Department Employee Count

**Normalized**
```sql
SELECT dept_id, COUNT(*) as emp_count
FROM employee
GROUP BY dept_id;
-- Scans entire table each query
```

**Denormalized**
```sql
CREATE TABLE department (
  dept_id INT PRIMARY KEY,
  dept_name VARCHAR(100),
  emp_count INT  -- Pre-calculated count
);

-- Instant result:
SELECT dept_id, dept_name, emp_count FROM department;
```

**Trade-off:**
- ✅ Instant result
- ❌ Must update emp_count when employees are added/removed

### 4. Storage of Derived References

Keep frequently-accessed references in main table.

#### Example: Product Category in Product Table

**Normalized (3NF)**
```sql
-- To find products by category name:
SELECT p.product_id, p.name
FROM product p
JOIN category c ON p.category_id = c.category_id
WHERE c.category_name = 'Electronics';
```

**Denormalized**
```sql
CREATE TABLE product_denorm (
  product_id INT PRIMARY KEY,
  name VARCHAR(100),
  category_id INT,
  category_name VARCHAR(100),  -- Duplicate from category table
  category_desc TEXT           -- Also duplicate
);

-- Now simpler:
SELECT product_id, name FROM product_denorm 
WHERE category_name = 'Electronics';
```

## Real-World Denormalization Examples

### Example 1: E-commerce Product Listing

**Normalized Approach**
```
Product → Category → Category Name
Product → Reviews → Review Count
Product → Inventory → Stock Count
```

**Multiple JOINs needed for product listing:**
```sql
SELECT p.product_id, p.name, c.category_name, 
       COUNT(r.review_id) as review_count,
       i.quantity_in_stock
FROM product p
JOIN category c ON p.category_id = c.category_id
LEFT JOIN review r ON p.product_id = r.product_id
LEFT JOIN inventory i ON p.product_id = i.product_id
GROUP BY p.product_id;
```

**Denormalized Approach**
```sql
CREATE TABLE product_listing (
  product_id INT PRIMARY KEY,
  name VARCHAR(100),
  category_id INT,
  category_name VARCHAR(100),    -- Denormalized
  price DECIMAL(10, 2),
  review_count INT,              -- Denormalized (pre-counted)
  quantity_in_stock INT,         -- Denormalized
  last_updated TIMESTAMP
);

-- Much simpler query:
SELECT * FROM product_listing;
```

**Why Denormalize?**
- Product listing page needs this data often
- Customers view products much more than edit them
- Reviews and inventory change less frequently

### Example 2: Dashboard with User Statistics

**Denormalized Data Warehouse**
```sql
CREATE TABLE user_stats_daily (
  user_id INT,
  date DATE,
  total_orders INT,          -- Denormalized (pre-calculated)
  total_spent DECIMAL(10, 2),-- Denormalized
  last_order_date DATE,      -- Denormalized
  loyalty_points INT,        -- Denormalized
  PRIMARY KEY (user_id, date)
);
```

**Why?** Dashboards read this data constantly. Users won't see updates immediately (batch updated daily). Performance is critical.

## Maintenance Strategies

### Strategy 1: Update Triggers

```sql
CREATE TRIGGER update_order_total 
AFTER INSERT ON order_item
FOR EACH ROW
BEGIN
  UPDATE orders 
  SET total_amount = (
    SELECT SUM(quantity * unit_price) 
    FROM order_item 
    WHERE order_id = NEW.order_id
  )
  WHERE order_id = NEW.order_id;
END;
```

### Strategy 2: Batch Updates

```sql
-- Nightly batch job to recalculate
UPDATE product_listing pl
SET review_count = (
  SELECT COUNT(*) FROM review r 
  WHERE r.product_id = pl.product_id
),
qty_in_stock = (
  SELECT SUM(quantity) FROM inventory i 
  WHERE i.product_id = pl.product_id
);
```

### Strategy 3: Application Layer

Handle denormalized data updates in application code:

```python
# When updating customer name:
1. Update customer table
2. Update customer_name in orders table
3. Update customer_name in invoices table
# Application ensures consistency
```

### Strategy 4: Caching Layer

```python
# Store denormalized data in cache:
customer_with_orders = {
  'customer_id': 1,
  'name': 'Alice',
  'email': 'alice@example.com',
  'total_orders': 5,           # Cached
  'lifetime_value': 5000       # Cached
}
# Cache invalidated when customer updates
```

## When Denormalization Goes Wrong

### Problem 1: Inconsistent Data

```
Customer name = "Alice" in customer table
Customer name = "John" in orders table

Same customer, different names! ❌
```

### Problem 2: Hard to Maintain

```sql
-- Update customer name:
UPDATE customer SET name = 'Alice' WHERE customer_id = 1;

-- Oops, forgot to update orders:
UPDATE orders SET customer_name = 'Alice' WHERE customer_id = 1;

-- Now some records have old name!
```

### Problem 3: Disk Space

```
1 million products × 5 denormalized fields = 5+ million extra data
```

## Denormalization in Different Systems

### OLTP (Online Transaction Processing)
Databases like MySQL used for applications:
- **Mostly normalized** (3NF usually)
- Denormalize selectively for critical queries
- Prioritize consistency

### OLAP (Online Analytical Processing)
Data warehouses used for analysis:
- **Heavily denormalized** (star/snowflake schemas)
- Performance critical
- Consistency less critical (batch loaded)
- Read-only or read-mostly

## Decision Matrix

| Scenario | Approach |
|----------|----------|
| Financial transactions | Stay normalized |
| User list | Can denormalize (static data) |
| Product search | Denormalize (read-heavy) |
| Inventory system | Stay normalized (frequent updates) |
| Analytics dashboard | Heavily denormalize |
| Audit log | Can denormalize (append-only) |

## Key Takeaways

✅ Denormalization = Intentional redundancy for performance
✅ Only denormalize when performance critical
✅ Prioritize read-heavy systems
✅ Be ready to manage consistency
✅ Use triggers or batch jobs to keep data in sync
✅ Document why data is denormalized

## Anti-Pattern: Premature Denormalization

❌ **Don't denormalize "just in case"**
- Start normalized (3NF)
- Only denormalize when you have performance data
- Profile queries first
- Measure impact

## Next Step

Learn about **[Database Applications](15-database-applications.md)** - Real-world usage patterns and best practices.
