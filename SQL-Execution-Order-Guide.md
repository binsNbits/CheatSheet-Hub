# SQL Execution Order Guide

A comprehensive guide to understanding SQL execution order, from script-level dependencies to clause-level syntax.

---

## Table of Contents

1. [Script Execution Order (Dependency Order)](#script-execution-order-dependency-order)
2. [SQL Statement Categories](#sql-statement-categories)
3. [Database Migration Sequence](#database-migration-sequence)
4. [SQL Clause Order (Lexical Syntax)](#sql-clause-order-lexical-syntax)
5. [Logical vs Lexical Execution Order](#logical-vs-lexical-execution-order)
6. [Common Patterns and Best Practices](#common-patterns-and-best-practices)

---

## Script Execution Order (Dependency Order)

### The Golden Rule

**You must BUILD the house (DDL) before you can MOVE FURNITURE into it (DML).**

### Understanding Script-Level Execution

When you execute a SQL script file containing multiple statements, they run **sequentially from top to bottom**. The order is dictated by **logical dependencies** - you cannot manipulate data in an object that doesn't exist yet.

### Typical Script Execution Sequence

```
1. DDL (Data Definition Language)    → Build the structure
2. DML (Data Manipulation Language)   → Populate with data
3. DQL (Data Query Language)          → Read and analyze data
4. DCL (Data Control Language)        → Set permissions
5. TCL (Transaction Control Language) → Manage transactions
```

### Detailed Execution Order

#### Phase 1: Structure Creation (DDL)

```sql
-- 1. Create schemas/databases first
CREATE SCHEMA company_db;

-- 2. Create tables (parent tables before child tables)
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);

-- 3. Create dependent tables (foreign keys require parent tables to exist)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- 4. Create indexes (after tables exist)
CREATE INDEX idx_user_email ON users(email);

-- 5. Create views (after all referenced tables exist)
CREATE VIEW user_order_summary AS
SELECT u.username, COUNT(o.order_id) as order_count
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.username;

-- 6. Create stored procedures/functions (after all dependencies exist)
CREATE PROCEDURE GetUserOrders(IN userId INT)
BEGIN
    SELECT * FROM orders WHERE user_id = userId;
END;

-- 7. Create triggers (after tables exist)
CREATE TRIGGER before_user_insert
BEFORE INSERT ON users
FOR EACH ROW
SET NEW.created_at = NOW();
```

#### Phase 2: Data Population (DML)

```sql
-- 8. Insert data (parent tables first, then child tables)
INSERT INTO users (user_id, username, email)
VALUES (1, 'john_doe', 'john@example.com');

INSERT INTO users (user_id, username, email)
VALUES (2, 'jane_smith', 'jane@example.com');

-- 9. Insert dependent data (after parent data exists)
INSERT INTO orders (order_id, user_id, order_date)
VALUES (101, 1, '2024-01-15');

-- 10. Update data if needed
UPDATE users SET email = 'newemail@example.com' WHERE user_id = 1;
```

#### Phase 3: Permissions (DCL)

```sql
-- 11. Grant permissions (after objects exist)
GRANT SELECT ON users TO 'read_only_user';
GRANT INSERT, UPDATE ON orders TO 'data_entry_user';
```

#### Phase 4: Transaction Control (TCL)

```sql
-- 12. Commit or rollback transactions
COMMIT;
-- or
ROLLBACK;
```

### Dependency Hierarchy

```
Level 1: SCHEMA/DATABASE
         ↓
Level 2: TABLES (Parent tables first)
         ↓
Level 3: TABLES (Child tables with foreign keys)
         ↓
Level 4: INDEXES, CONSTRAINTS
         ↓
Level 5: VIEWS (simple views)
         ↓
Level 6: VIEWS (complex views referencing other views)
         ↓
Level 7: STORED PROCEDURES, FUNCTIONS
         ↓
Level 8: TRIGGERS
         ↓
Level 9: DATA INSERTION (Parent → Child)
         ↓
Level 10: PERMISSIONS
```

---

## SQL Statement Categories

### DDL (Data Definition Language)
**Purpose**: Define and modify database structure

```sql
CREATE   -- Create new database objects
ALTER    -- Modify existing objects
DROP     -- Delete objects
TRUNCATE -- Remove all data from table (keeps structure)
RENAME   -- Rename objects
```

### DML (Data Manipulation Language)
**Purpose**: Manipulate data within objects

```sql
INSERT   -- Add new records
UPDATE   -- Modify existing records
DELETE   -- Remove records
MERGE    -- Insert or update (upsert)
```

### DQL (Data Query Language)
**Purpose**: Retrieve and analyze data

```sql
SELECT   -- Query data
```

### DCL (Data Control Language)
**Purpose**: Control access to data

```sql
GRANT    -- Give permissions
REVOKE   -- Remove permissions
```

### TCL (Transaction Control Language)
**Purpose**: Manage transactions

```sql
BEGIN TRANSACTION  -- Start transaction
COMMIT            -- Save changes
ROLLBACK          -- Undo changes
SAVEPOINT         -- Create restore point
```

---

## Database Migration Sequence

### Migration File Naming Convention

```
001_create_users_table.sql
002_create_orders_table.sql
003_add_email_index.sql
004_create_products_table.sql
005_add_foreign_keys.sql
006_seed_initial_data.sql
```

### Migration Best Practices

1. **Incremental Changes**: Each migration should be atomic and reversible
2. **Version Control**: Number migrations sequentially
3. **Idempotent Scripts**: Use `IF NOT EXISTS` to prevent errors on re-run
4. **Rollback Scripts**: Create corresponding down migrations

### Example Migration Pattern

```sql
-- UP Migration (001_create_users.sql)
CREATE TABLE IF NOT EXISTS users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- DOWN Migration (001_create_users_down.sql)
DROP TABLE IF EXISTS users;
```

### Complex Migration Order

```sql
-- Migration 1: Create independent tables
CREATE TABLE users (...);
CREATE TABLE products (...);

-- Migration 2: Create relationship tables
CREATE TABLE user_products (
    user_id INT,
    product_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

-- Migration 3: Add indexes for performance
CREATE INDEX idx_user_id ON user_products(user_id);
CREATE INDEX idx_product_id ON user_products(product_id);

-- Migration 4: Add constraints
ALTER TABLE users ADD CONSTRAINT unique_username UNIQUE (username);

-- Migration 5: Seed reference data
INSERT INTO products (name, type) VALUES ('Default', 'system');

-- Migration 6: Create computed views
CREATE VIEW user_product_summary AS ...;
```

---

## SQL Clause Order (Lexical Syntax)

### The Critical Distinction

- **Lexical Order**: The order you MUST write clauses (syntax requirement)
- **Logical Order**: The order the database engine ACTUALLY executes them

### SELECT Statement - Lexical Order (How You Write It)

```sql
SELECT      -- 1. Specify columns to return
FROM        -- 2. Specify source table(s)
JOIN        -- 3. Join additional tables
WHERE       -- 4. Filter rows BEFORE grouping
GROUP BY    -- 5. Group rows for aggregation
HAVING      -- 6. Filter groups AFTER aggregation
ORDER BY    -- 7. Sort the result set
LIMIT       -- 8. Restrict number of rows returned
OFFSET      -- 9. Skip rows (pagination)
```

**Mnemonic**: **S**o **F**ew **J**udges **W**ill **G**ive **H**igh **O**rders **L**ately

### SELECT Statement - Complete Syntax

```sql
SELECT [DISTINCT] column1, column2, aggregate_function(column3)
FROM table1
[INNER|LEFT|RIGHT|FULL] JOIN table2 ON table1.id = table2.foreign_id
WHERE condition1 AND condition2
GROUP BY column1, column2
HAVING aggregate_condition
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC]
LIMIT row_count
OFFSET skip_count;
```

### Example with All Clauses

```sql
SELECT
    department,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary
FROM employees
INNER JOIN departments ON employees.dept_id = departments.id
WHERE hire_date >= '2020-01-01'
    AND status = 'active'
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY avg_salary DESC
LIMIT 10
OFFSET 0;
```

### INSERT Statement - Lexical Order

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);

-- Or with SELECT
INSERT INTO table_name (column1, column2)
SELECT column1, column2
FROM other_table
WHERE condition;
```

### UPDATE Statement - Lexical Order

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition
ORDER BY column  -- (MySQL only)
LIMIT row_count; -- (MySQL only)
```

### DELETE Statement - Lexical Order

```sql
DELETE FROM table_name
WHERE condition
ORDER BY column  -- (MySQL only)
LIMIT row_count; -- (MySQL only)
```

### CREATE TABLE Statement - Lexical Order

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column1 datatype [constraints],
    column2 datatype [constraints],
    ...
    [table_constraints]
) [table_options];
```

**Column Constraint Order**:
```sql
column_name
    datatype
    [NOT NULL]
    [UNIQUE]
    [PRIMARY KEY]
    [FOREIGN KEY REFERENCES table(column)]
    [CHECK (condition)]
    [DEFAULT value]
    [AUTO_INCREMENT]
```

### ALTER TABLE Statement - Lexical Order

```sql
ALTER TABLE table_name
    [ADD column_definition]
    [DROP COLUMN column_name]
    [MODIFY COLUMN column_definition]
    [ADD CONSTRAINT constraint_definition]
    [DROP CONSTRAINT constraint_name];
```

---

## Logical vs Lexical Execution Order

### The SELECT Statement Paradox

You **write** a SELECT query in one order (lexical), but the database **executes** it in a completely different order (logical).

### Logical Execution Order (How Database Processes It)

```
1. FROM       → Identify source tables
2. JOIN       → Combine tables
3. WHERE      → Filter individual rows
4. GROUP BY   → Aggregate rows into groups
5. HAVING     → Filter aggregated groups
6. SELECT     → Determine which columns to return
7. DISTINCT   → Remove duplicate rows
8. ORDER BY   → Sort the result set
9. LIMIT      → Restrict row count
10. OFFSET    → Skip rows
```

### Why This Matters

Understanding logical order explains:

- **Why you can't use column aliases in WHERE**: The SELECT hasn't executed yet!
- **Why HAVING comes after GROUP BY**: It filters groups, not rows
- **Why ORDER BY can use aliases**: SELECT has already executed

### Example Demonstrating the Difference

```sql
-- This WORKS
SELECT
    salary * 1.1 AS new_salary  -- Executed at step 6
FROM employees                   -- Executed at step 1
WHERE salary > 50000             -- Executed at step 3 (before SELECT)
ORDER BY new_salary;             -- Executed at step 8 (after SELECT, alias available)

-- This FAILS
SELECT
    salary * 1.1 AS new_salary
FROM employees
WHERE new_salary > 50000;  -- ERROR! Alias not available yet
                           -- WHERE executes before SELECT
```

### Visual Diagram

```
LEXICAL ORDER (How you write):
┌────────────────────────────────────────┐
│ SELECT → FROM → WHERE → GROUP BY       │
│   → HAVING → ORDER BY → LIMIT          │
└────────────────────────────────────────┘

LOGICAL ORDER (How it executes):
┌────────────────────────────────────────┐
│ FROM → WHERE → GROUP BY → HAVING       │
│   → SELECT → ORDER BY → LIMIT          │
└────────────────────────────────────────┘
```

---

## Common Patterns and Best Practices

### Pattern 1: Script File Organization

```sql
-- ============================================
-- DATABASE SETUP SCRIPT
-- Version: 1.0
-- Author: Your Name
-- Date: 2024-01-15
-- ============================================

-- Step 1: Create Database
CREATE DATABASE IF NOT EXISTS myapp_db;
USE myapp_db;

-- Step 2: Create Tables (Dependency Order)
-- No foreign keys first
CREATE TABLE roles (...);
CREATE TABLE users (...);

-- Tables with foreign keys second
CREATE TABLE user_roles (...);
CREATE TABLE posts (...);

-- Step 3: Create Indexes
CREATE INDEX idx_username ON users(username);

-- Step 4: Create Views
CREATE VIEW active_users AS ...;

-- Step 5: Insert Initial Data
INSERT INTO roles VALUES (1, 'Admin'), (2, 'User');

-- Step 6: Set Permissions
GRANT SELECT ON myapp_db.* TO 'app_reader';

-- Step 7: Commit
COMMIT;
```

### Pattern 2: Complex JOIN Query Structure

```sql
SELECT
    -- Aggregations and calculations in SELECT
    u.username,
    COUNT(DISTINCT o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent,
    AVG(o.total_amount) as avg_order_value
FROM
    users u                                    -- Primary table
    INNER JOIN orders o                        -- Required relationship
        ON u.user_id = o.user_id
    LEFT JOIN user_preferences up              -- Optional data
        ON u.user_id = up.user_id
WHERE
    u.status = 'active'                        -- Filter before grouping
    AND o.order_date >= DATE_SUB(NOW(), INTERVAL 1 YEAR)
GROUP BY
    u.user_id,                                 -- Group by user
    u.username
HAVING
    COUNT(DISTINCT o.order_id) > 5             -- Filter groups
ORDER BY
    total_spent DESC                           -- Sort by calculated column
LIMIT 100;
```

### Pattern 3: Subquery Positioning

```sql
-- Subquery in FROM (derived table)
SELECT avg_salary
FROM (
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
) AS dept_averages
WHERE avg_salary > 50000;

-- Subquery in WHERE (filtering)
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'NYC'
);

-- Subquery in SELECT (correlated)
SELECT
    name,
    salary,
    (SELECT AVG(salary) FROM employees e2
     WHERE e2.department_id = e1.department_id) as dept_avg
FROM employees e1;
```

### Pattern 4: Transaction Structure

```sql
-- Begin transaction
START TRANSACTION;

-- Multiple related operations
INSERT INTO orders (user_id, total) VALUES (1, 100.00);
SET @order_id = LAST_INSERT_ID();

INSERT INTO order_items (order_id, product_id, quantity)
VALUES (@order_id, 101, 2);

UPDATE inventory
SET quantity = quantity - 2
WHERE product_id = 101;

-- Check if everything worked
-- If any statement fails, ROLLBACK
-- If all succeed, COMMIT
COMMIT;
```

### Pattern 5: Safe Schema Changes

```sql
-- Always check existence before creating
CREATE TABLE IF NOT EXISTS new_table (...);

-- Safe column addition
ALTER TABLE users
ADD COLUMN IF NOT EXISTS phone VARCHAR(20);

-- Safe index creation
CREATE INDEX IF NOT EXISTS idx_email ON users(email);

-- Drop with safety check
DROP TABLE IF EXISTS temp_table;
```

### Common Mistakes to Avoid

#### Mistake 1: Wrong Clause Order
```sql
-- WRONG - Syntax Error
SELECT *
WHERE status = 'active'
FROM users;

-- CORRECT
SELECT *
FROM users
WHERE status = 'active';
```

#### Mistake 2: Using Aliases Too Early
```sql
-- WRONG - Can't use alias in WHERE
SELECT salary * 1.1 AS new_salary
FROM employees
WHERE new_salary > 50000;

-- CORRECT - Use original expression
SELECT salary * 1.1 AS new_salary
FROM employees
WHERE salary * 1.1 > 50000;

-- OR use subquery
SELECT * FROM (
    SELECT salary * 1.1 AS new_salary
    FROM employees
) AS derived
WHERE new_salary > 50000;
```

#### Mistake 3: Wrong Table Creation Order
```sql
-- WRONG - Child before parent
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
-- ERROR: Table 'users' doesn't exist

CREATE TABLE users (...);

-- CORRECT - Parent before child
CREATE TABLE users (
    user_id INT PRIMARY KEY
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

#### Mistake 4: HAVING vs WHERE
```sql
-- WRONG - Can't use aggregate in WHERE
SELECT department, COUNT(*) as emp_count
FROM employees
WHERE COUNT(*) > 5  -- ERROR
GROUP BY department;

-- CORRECT - Use HAVING for aggregates
SELECT department, COUNT(*) as emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

---

## Quick Reference Tables

### Script Execution Order
| Order | Category | Purpose | Example |
|-------|----------|---------|---------|
| 1 | DDL | Create structure | CREATE TABLE |
| 2 | DML | Populate data | INSERT INTO |
| 3 | DQL | Query data | SELECT |
| 4 | DCL | Set permissions | GRANT |
| 5 | TCL | Control transactions | COMMIT |

### SELECT Clause Order
| Lexical Order | Logical Order | Clause | Purpose |
|---------------|---------------|--------|---------|
| 1 | 6 | SELECT | Choose columns |
| 2 | 1 | FROM | Specify table |
| 3 | 2 | JOIN | Combine tables |
| 4 | 3 | WHERE | Filter rows |
| 5 | 4 | GROUP BY | Aggregate rows |
| 6 | 5 | HAVING | Filter groups |
| 7 | 7 | ORDER BY | Sort results |
| 8 | 8 | LIMIT | Restrict count |

### Statement Type Clause Order
| Statement | Required Clauses | Optional Clauses |
|-----------|-----------------|------------------|
| SELECT | SELECT, FROM | JOIN, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT |
| INSERT | INSERT INTO, VALUES/SELECT | - |
| UPDATE | UPDATE, SET | WHERE, ORDER BY, LIMIT |
| DELETE | DELETE FROM | WHERE, ORDER BY, LIMIT |
| CREATE TABLE | CREATE TABLE, column_definitions | constraints, options |
| ALTER TABLE | ALTER TABLE | ADD, DROP, MODIFY |

---

## Conclusion

Understanding SQL execution order at both the **script level** (dependency order) and **statement level** (clause order) is fundamental to writing correct, efficient SQL code.

**Key Takeaways:**

1. **Script Level**: Follow dependency order - create objects before using them
2. **Statement Level**: Follow lexical syntax - write clauses in required order
3. **Execution Level**: Understand logical order - know when aliases are available
4. **Migration Level**: Use sequential, versioned, reversible changes

**Remember:**
- Build the house (DDL) before moving in furniture (DML)
- Write clauses in lexical order, understand logical execution order
- Parent tables before child tables
- Structure before data
- Data before permissions

---

*Last Updated: 2024-01-15*
