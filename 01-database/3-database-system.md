# Database Systems

## What is a Database System?

A **Database System** (or DBMS - Database Management System) is **software** that manages databases. It handles storage, retrieval, and modification of data.

## Components of a Database System

```
┌─────────────────────────────────────┐
│     Application Layer               │
│   (Your web app, mobile app, etc)   │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│   Database Management System         │
│  ┌──────────────────────────────┐   │
│  │  Query Processor/Optimizer   │   │
│  │  (Interprets SQL)            │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │  Transaction Manager         │   │
│  │  (ACID compliance)           │   │
│  └──────────────────────────────┘   │
│  ┌──────────────────────────────┐   │
│  │  Storage Engine              │   │
│  │  (Reads/writes to disk)      │   │
│  └──────────────────────────────┘   │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│  Physical Storage (Hard Disk/SSD)   │
└─────────────────────────────────────┘
```

## DBMS vs RDBMS

### DBMS (General)
- Any software that manages a database
- Can be relational or non-relational
- Examples: MongoDB (NoSQL), Redis, File-based systems

### RDBMS (Relational)
- **Specific type** of DBMS for relational databases
- Organizes data in **tables with relationships**
- Enforces **ACID properties**
- Supports **SQL language**
- Examples: MySQL, PostgreSQL, Oracle, SQL Server

## RDBMS Architecture

### Query Processing Pipeline

1. **SQL Parsing**
   - Check syntax correctness
   - Validate table/column names

2. **Optimization**
   - Determine best way to execute query
   - Choose indexes to use
   - Estimate resource requirements

3. **Compilation**
   - Convert to internal format
   - Generate execution plan

4. **Execution**
   - Execute on storage engine
   - Return results

## Key Responsibilities of a DBMS

### 1. Data Storage
- Organize data efficiently
- Use indexes for fast retrieval
- Manage disk space

### 2. Data Retrieval
- Execute queries
- Apply filters and joins
- Optimize performance

### 3. Data Modification
- Insert new records
- Update existing records
- Delete records safely

### 4. Data Integrity
- Enforce constraints
- Prevent invalid data
- Maintain relationships

### 5. Concurrency Control
- Allow multiple users simultaneously
- Prevent conflicts
- Lock management

### 6. Backup & Recovery
- Create backups
- Restore from failures
- Transaction logs

### 7. Security
- User authentication
- Permission management
- Encryption

## MySQL: An RDBMS Example

MySQL is an **open-source RDBMS** that:

- ✅ Runs on Linux, Windows, macOS
- ✅ Uses SQL for queries
- ✅ Provides ACID transactions
- ✅ Supports replication and clustering
- ✅ Free and widely adopted
- ✅ Used by Facebook, Twitter, YouTube, etc.

### MySQL Architecture

```
MySQL Server
├── Connection Layer
│   ├── Authentication
│   └── Connection Pool
├── Query Layer
│   ├── Parser
│   ├── Optimizer
│   └── Cache
└── Storage Layer
    ├── InnoDB (default, ACID)
    ├── MyISAM (fast, no transactions)
    └── Other engines
```

## Transaction ACID Properties

All RDBMSs must guarantee ACID:

### **A**tomicity
- Transaction completes fully or not at all
- No partial updates

### **C**onsistency
- Database moves from one valid state to another
- All rules are enforced

### **I**solation
- Concurrent transactions don't interfere
- Each transaction is independent

### **D**urability
- Once committed, data survives failures
- Permanent storage

## Real-World Database Operations

### Example: Bank Transfer

**Without proper DBMS:**
```
Account A balance = 1000
Account B balance = 500

Debit $100 from A → A = 900
System crashes before...
Credit $100 to B → B = 500 (never happens!)
Result: $100 lost!
```

**With ACID DBMS:**
```
BEGIN TRANSACTION
  UPDATE accounts SET balance = balance - 100 WHERE id = A
  UPDATE accounts SET balance = balance + 100 WHERE id = B
COMMIT (or ROLLBACK if error)
```
Either both succeed or both fail. Money never disappears!

## Key Takeaways

✅ DBMS = Software that manages databases
✅ RDBMS = Specialized DBMS for relational data
✅ MySQL is a popular, powerful open-source RDBMS
✅ ACID guarantees = Data reliability
✅ DBMS handles queries, concurrency, security, backups

## Next Step

Learn about **[Data Modeling](4-data-modeling.md)** - how to design your database structure.
