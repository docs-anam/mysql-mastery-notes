# Attributes and Data Types

## What are Attributes?

**Attributes** are properties or characteristics of an entity that you want to store.

### Example: Customer Entity
```
Customer Attributes:
- Name (text)
- Email (text)
- Phone (text)
- Birth Date (date)
- Customer Since (date)
- Is Active (yes/no)
- Loyalty Points (number)
- Address (text)
```

## Data Types in MySQL

When you define attributes, you must choose a **data type** that specifies:
- What kind of data it stores
- How much storage it uses
- What operations are allowed

## String Data Types

### CHAR
- **Fixed length** string
- **Size**: Up to 255 characters
- **Usage**: Fixed-width data (postal codes, country codes)
- **Storage**: Always uses specified size
- **Speed**: Slightly faster than VARCHAR

```sql
CREATE TABLE example (
  country_code CHAR(2)  -- Always exactly 2 characters
);
```

### VARCHAR
- **Variable length** string
- **Size**: Up to 65,535 characters
- **Usage**: Most common for text data
- **Storage**: Uses only needed space
- **Speed**: Slightly slower than CHAR, but more efficient

```sql
CREATE TABLE example (
  email VARCHAR(100),  -- Up to 100 characters
  name VARCHAR(255)    -- Up to 255 characters
);
```

### TEXT
- **Large text** data
- **Size**: Up to 65,535 characters
- **Usage**: Articles, descriptions, comments
- **Storage**: Stored separately from row

```sql
CREATE TABLE example (
  description TEXT,   -- For long descriptions
  bio TEXT           -- For user bios
);
```

### ENUM
- **Fixed set of values**
- **Size**: List of allowed options
- **Usage**: Status, type, category with limited choices
- **Efficient**: Stores as number internally

```sql
CREATE TABLE example (
  status ENUM('active', 'inactive', 'suspended'),
  role ENUM('admin', 'user', 'guest')
);
```

## Numeric Data Types

### TINYINT
- **Size**: 1 byte
- **Range**: -128 to 127 (or 0 to 255 unsigned)
- **Usage**: Small counts, flags, percentages

### SMALLINT
- **Size**: 2 bytes
- **Range**: -32,768 to 32,767
- **Usage**: Larger counts, ages, quantities

### INT (INTEGER)
- **Size**: 4 bytes
- **Range**: -2,147,483,648 to 2,147,483,647
- **Usage**: IDs, counts, normal numbers
- **Most common** for general numbers

### BIGINT
- **Size**: 8 bytes
- **Range**: Very large numbers
- **Usage**: Large IDs, timestamps in milliseconds

### DECIMAL (NUMERIC)
- **Size**: Variable
- **Precision**: Fixed decimal places
- **Usage**: **Always use for money!**
- **Why**: Avoids floating-point rounding errors

```sql
CREATE TABLE product (
  price DECIMAL(10, 2)  -- 10 total digits, 2 after decimal
);
-- $999,999.99 is maximum
```

### FLOAT / DOUBLE
- **Floating point** numbers
- **Approximate** values
- **Usage**: Scientific calculations, rarely for money
- **Warning**: Has rounding errors

## Date and Time Data Types

### DATE
- **Format**: YYYY-MM-DD
- **Size**: 3 bytes
- **Usage**: Birth dates, event dates
- **Range**: '1000-01-01' to '9999-12-31'

```sql
CREATE TABLE customer (
  birth_date DATE
);
```

### TIME
- **Format**: HH:MM:SS
- **Size**: 3 bytes
- **Usage**: Time of day
- **Range**: '-838:59:59' to '838:59:59'

### DATETIME
- **Format**: YYYY-MM-DD HH:MM:SS
- **Size**: 8 bytes
- **Usage**: Specific moment (post created, order placed)
- **No timezone** support

```sql
CREATE TABLE orders (
  order_datetime DATETIME  -- 2024-02-11 14:30:45
);
```

### TIMESTAMP
- **Format**: YYYY-MM-DD HH:MM:SS
- **Size**: 4 bytes
- **Usage**: System timestamps, automatic updates
- **Timezone aware**: Converted to UTC storage
- **Auto-update**: Can auto-update on row change

```sql
CREATE TABLE post (
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### YEAR
- **Format**: YYYY
- **Size**: 1 byte
- **Usage**: Year values
- **Range**: 1901 to 2155

## Boolean Data Types

### BOOLEAN (BOOL)
- **Actually**: TINYINT(1)
- **Values**: TRUE (1) or FALSE (0)
- **Size**: 1 byte
- **Usage**: Flags, yes/no fields

```sql
CREATE TABLE customer (
  is_active BOOLEAN DEFAULT TRUE,
  is_vip BOOLEAN DEFAULT FALSE
);
```

## Binary Data Types

### BLOB
- **Binary Large Object**
- **Usage**: Images, files, binary data
- **Size**: Up to 65,535 bytes
- **Note**: Usually store in file system, not database

## JSON Data Type

### JSON
- **Stores JSON documents**
- **Size**: Depends on content
- **Usage**: Flexible, semi-structured data
- **Features**: Query and validate JSON

```sql
CREATE TABLE user (
  settings JSON  -- Can store {"theme": "dark", "language": "en"}
);
```

## Choosing the Right Data Type

### For Text
- **Fixed length**: Use CHAR
- **Variable, short**: Use VARCHAR(n)
- **Long text**: Use TEXT
- **Limited options**: Use ENUM

### For Numbers
- **Small counts**: TINYINT or SMALLINT
- **IDs and counts**: INT
- **Large numbers**: BIGINT
- **Money**: DECIMAL(10, 2) - NEVER FLOAT
- **Scientific**: FLOAT or DOUBLE

### For Dates/Times
- **Date only**: DATE
- **Time only**: TIME
- **Specific moment**: DATETIME or TIMESTAMP
- **Year only**: YEAR

## Storage and Performance

| Type | Storage | Use |
|------|---------|-----|
| CHAR(20) | Always 20 bytes | Fixed-width, frequently searched |
| VARCHAR(20) | 1-20 bytes | Variable-width, saves space |
| INT | 4 bytes | Numeric IDs |
| DECIMAL(10,2) | 5 bytes | Money - accuracy matters |
| FLOAT | 4 bytes | Approximate numbers |
| DATE | 3 bytes | Dates only |
| TIMESTAMP | 4 bytes | System timestamps |

## Best Practices

✅ **Use appropriate types** - Saves space and improves performance
✅ **Use DECIMAL for money** - NEVER FLOAT
✅ **Use VARCHAR with reasonable limits** - VARCHAR(255) for email
✅ **Use ENUM for limited choices** - Saves space, enforces values
✅ **Use TIMESTAMP for system dates** - Auto-updates
✅ **Use NOT NULL for required fields** - Prevents NULL confusion

## Example: E-commerce Product Table

```sql
CREATE TABLE product (
  product_id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  sku CHAR(10) UNIQUE NOT NULL,
  category ENUM('Electronics', 'Books', 'Clothing') NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  discount_percent TINYINT DEFAULT 0,
  stock_quantity INT DEFAULT 0,
  rating DECIMAL(3, 2),
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## Key Takeaways

✅ Choose data types that match your data
✅ Use DECIMAL for money, not FLOAT
✅ VARCHAR saves space for variable-length text
✅ ENUM enforces limited choices
✅ TIMESTAMP auto-updates, great for system fields
✅ Right types = Better performance + Less storage

## Next Step

Learn about **[Keys and Constraints](6-attribute-key.md)** - How to identify unique records.
