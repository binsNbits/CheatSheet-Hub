# SQL Complete Developer Reference

The ultimate comprehensive guide for SQL development - covering execution order, all SQL categories, PostgreSQL specifics, procedural programming, and advanced features.

---

## Table of Contents

### Part I: Fundamentals
1. [Quick Reference](#quick-reference)
2. [Execution Order Guide](#execution-order-guide)
3. [SQL Statement Categories Overview](#sql-statement-categories-overview)

### Part II: Core SQL Commands
4. [DDL - Data Definition Language](#ddl---data-definition-language)
5. [DQL - Data Query Language](#dql---data-query-language)
6. [DML - Data Manipulation Language](#dml---data-manipulation-language)
7. [DCL - Data Control Language](#dcl---data-control-language)
8. [TCL - Transaction Control Language](#tcl---transaction-control-language)

### Part III: PostgreSQL Specifics
9. [PostgreSQL Connection & Session](#postgresql-connection--session)
10. [psql Meta-Commands](#psql-meta-commands)

### Part IV: Procedural SQL
11. [Variables and Control Flow](#variables-and-control-flow)
12. [Functions and Stored Procedures](#functions-and-stored-procedures)
13. [Triggers](#triggers)
14. [Cursors](#cursors)
15. [Exception Handling](#exception-handling)

### Part V: Advanced Features
16. [Window Functions](#window-functions)
17. [Common Table Expressions (CTEs)](#common-table-expressions-ctes)
18. [JSON Operations](#json-operations)
19. [Full-Text Search](#full-text-search)
20. [Partitioning](#partitioning)
21. [Indexes and Performance](#indexes-and-performance)

### Part VI: Best Practices
22. [Common Patterns](#common-patterns)
23. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
24. [Database Migration Best Practices](#database-migration-best-practices)

---

# Part I: Fundamentals

## Quick Reference

### The Golden Rules

1. **Script Level**: Build the house (DDL) before moving furniture (DML)
2. **Statement Level**: Write in lexical order, understand logical execution order
3. **Dependencies**: Parent tables before child tables, structure before data
4. **Aliases**: Can't use in WHERE (executes before SELECT), can use in ORDER BY

### SELECT Statement Order Mnemonic

**"So Few Judges Will Give High Orders Lately"**

```sql
SELECT      -- 1. Specify columns
FROM        -- 2. Source tables
JOIN        -- 3. Combine tables
WHERE       -- 4. Filter rows
GROUP BY    -- 5. Aggregate rows
HAVING      -- 6. Filter groups
ORDER BY    -- 7. Sort results
LIMIT       -- 8. Restrict count
```

### Statement Categories Quick Map

| Category | Purpose | Commands |
|----------|---------|----------|
| DDL | Define structure | CREATE, ALTER, DROP, TRUNCATE |
| DML | Manipulate data | INSERT, UPDATE, DELETE, MERGE |
| DQL | Query data | SELECT |
| DCL | Control access | GRANT, REVOKE |
| TCL | Manage transactions | BEGIN, COMMIT, ROLLBACK, SAVEPOINT |

---

## Execution Order Guide

### Script Execution Order (Dependency Order)

When you execute a SQL script file containing multiple statements, they run **sequentially from top to bottom**. The order is dictated by **logical dependencies**.

#### Dependency Hierarchy

```
Level 1:  SCHEMA/DATABASE
          ↓
Level 2:  TABLES (Parent tables first - no foreign keys)
          ↓
Level 3:  TABLES (Child tables with foreign keys)
          ↓
Level 4:  INDEXES, CONSTRAINTS
          ↓
Level 5:  VIEWS (simple views)
          ↓
Level 6:  VIEWS (complex views referencing other views)
          ↓
Level 7:  STORED PROCEDURES, FUNCTIONS
          ↓
Level 8:  TRIGGERS
          ↓
Level 9:  DATA INSERTION (Parent → Child)
          ↓
Level 10: PERMISSIONS (GRANT/REVOKE)
```

#### Complete Script Example

```sql
-- ============================================
-- DATABASE SETUP SCRIPT
-- ============================================

-- PHASE 1: STRUCTURE CREATION (DDL)
-- 1. Create database/schema
CREATE DATABASE IF NOT EXISTS company_db;
USE company_db;

-- 2. Create parent tables (no foreign keys)
CREATE TABLE departments (
    dept_id INT PRIMARY KEY AUTO_INCREMENT,
    dept_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE roles (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL
);

-- 3. Create child tables (with foreign keys)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    dept_id INT,
    manager_id INT,
    salary DECIMAL(10, 2) CHECK (salary > 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id) ON DELETE SET NULL,
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id) ON DELETE SET NULL
);

-- 4. Create indexes
CREATE INDEX idx_emp_email ON employees(email);
CREATE INDEX idx_emp_dept ON employees(dept_id);

-- 5. Create views
CREATE VIEW employee_summary AS
SELECT
    e.emp_id,
    e.first_name,
    e.last_name,
    d.dept_name,
    e.salary
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;

-- 6. Create stored procedures/functions
DELIMITER //
CREATE FUNCTION get_employee_count(dept INT) RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE emp_count INT;
    SELECT COUNT(*) INTO emp_count FROM employees WHERE dept_id = dept;
    RETURN emp_count;
END //
DELIMITER ;

-- 7. Create triggers
DELIMITER //
CREATE TRIGGER before_emp_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
END //
DELIMITER ;

-- PHASE 2: DATA POPULATION (DML)
-- 8. Insert parent data first
INSERT INTO departments (dept_name) VALUES
    ('Engineering'),
    ('Sales'),
    ('HR');

INSERT INTO roles (role_name) VALUES
    ('Manager'),
    ('Developer'),
    ('Analyst');

-- 9. Insert child data
INSERT INTO employees (first_name, last_name, email, dept_id, salary)
VALUES
    ('John', 'Doe', 'john@company.com', 1, 75000),
    ('Jane', 'Smith', 'jane@company.com', 2, 65000);

-- PHASE 3: PERMISSIONS (DCL)
-- 10. Grant permissions
GRANT SELECT ON company_db.* TO 'read_user'@'localhost';
GRANT SELECT, INSERT, UPDATE ON company_db.employees TO 'hr_user'@'localhost';

-- PHASE 4: TRANSACTION CONTROL (TCL)
-- 11. Commit changes
COMMIT;
```

### Statement Execution Order (Lexical vs Logical)

#### The SELECT Statement Paradox

You **write** a SELECT query in one order (lexical), but the database **executes** it in a completely different order (logical).

#### Lexical Order (How You Write It)

```sql
SELECT      -- 1. What columns to return
FROM        -- 2. Source table(s)
JOIN        -- 3. Join other tables
WHERE       -- 4. Filter rows
GROUP BY    -- 5. Group rows
HAVING      -- 6. Filter groups
ORDER BY    -- 7. Sort results
LIMIT       -- 8. Restrict count
OFFSET      -- 9. Skip rows
```

#### Logical Order (How Database Executes It)

```
1. FROM       → Identify and load source tables
2. JOIN       → Combine tables based on join conditions
3. WHERE      → Filter individual rows (before grouping)
4. GROUP BY   → Aggregate rows into groups
5. HAVING     → Filter aggregated groups
6. SELECT     → Determine which columns to return
7. DISTINCT   → Remove duplicate rows
8. ORDER BY   → Sort the result set
9. LIMIT      → Restrict number of rows
10. OFFSET    → Skip rows for pagination
```

#### Why This Matters

Understanding logical order explains common errors:

```sql
-- ❌ THIS FAILS - Alias not available in WHERE
SELECT salary * 1.1 AS new_salary
FROM employees
WHERE new_salary > 50000;  -- ERROR: WHERE executes before SELECT

-- ✅ THIS WORKS - Use original expression in WHERE
SELECT salary * 1.1 AS new_salary
FROM employees
WHERE salary * 1.1 > 50000;

-- ✅ ALTERNATIVE - Use subquery
SELECT * FROM (
    SELECT salary * 1.1 AS new_salary
    FROM employees
) AS derived
WHERE new_salary > 50000;

-- ✅ THIS WORKS - ORDER BY executes after SELECT
SELECT salary * 1.1 AS new_salary
FROM employees
WHERE salary > 50000
ORDER BY new_salary DESC;  -- Alias available here!
```

#### Visual Diagram

```
LEXICAL ORDER (How you write):
┌────────────────────────────────────────────────┐
│ SELECT → FROM → JOIN → WHERE → GROUP BY →     │
│ HAVING → ORDER BY → LIMIT → OFFSET            │
└────────────────────────────────────────────────┘

LOGICAL ORDER (How it executes):
┌────────────────────────────────────────────────┐
│ FROM → JOIN → WHERE → GROUP BY → HAVING →     │
│ SELECT → DISTINCT → ORDER BY → LIMIT → OFFSET │
└────────────────────────────────────────────────┘
```

---

## SQL Statement Categories Overview

### DDL (Data Definition Language)
**Purpose**: Define and modify database structure

```sql
CREATE      -- Create new database objects (tables, views, indexes)
ALTER       -- Modify existing objects
DROP        -- Delete objects permanently
TRUNCATE    -- Remove all data from table (keeps structure)
RENAME      -- Rename objects
```

### DML (Data Manipulation Language)
**Purpose**: Manipulate data within objects

```sql
INSERT      -- Add new records
UPDATE      -- Modify existing records
DELETE      -- Remove records
MERGE       -- Insert or update (upsert)
```

### DQL (Data Query Language)
**Purpose**: Retrieve and analyze data

```sql
SELECT      -- Query and retrieve data
```

### DCL (Data Control Language)
**Purpose**: Control access to data

```sql
GRANT       -- Give permissions to users
REVOKE      -- Remove permissions from users
```

### TCL (Transaction Control Language)
**Purpose**: Manage transactions

```sql
BEGIN TRANSACTION    -- Start a new transaction
COMMIT              -- Save all changes permanently
ROLLBACK            -- Undo all changes since BEGIN
SAVEPOINT           -- Create restore point within transaction
```

---

# Part II: Core SQL Commands

## DDL - Data Definition Language

### CREATE DATABASE

```sql
-- Basic syntax
CREATE DATABASE database_name;

-- With character set and collation (MySQL)
CREATE DATABASE database_name
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- PostgreSQL with encoding
CREATE DATABASE database_name
    ENCODING 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8';

-- Idempotent creation
CREATE DATABASE IF NOT EXISTS database_name;
```

### CREATE TABLE

#### Basic Syntax

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column1 datatype [constraints],
    column2 datatype [constraints],
    ...
    [table_constraints]
) [table_options];
```

#### Comprehensive Example

```sql
CREATE TABLE employees (
    -- Primary key (auto-increment)
    employee_id INT AUTO_INCREMENT,           -- MySQL
    -- employee_id SERIAL,                    -- PostgreSQL
    -- employee_id INT IDENTITY(1,1),         -- SQL Server

    -- Basic columns with constraints
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),

    -- Date/time columns
    hire_date DATE NOT NULL DEFAULT CURRENT_DATE,
    birth_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- Numeric columns
    salary DECIMAL(10, 2) CHECK (salary > 0),
    bonus DECIMAL(10, 2) DEFAULT 0.00,

    -- Foreign keys
    department_id INT,
    manager_id INT,

    -- Enum/Boolean
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    is_remote BOOLEAN DEFAULT FALSE,

    -- Primary key constraint
    PRIMARY KEY (employee_id),

    -- Foreign key constraints
    CONSTRAINT fk_department
        FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE,

    CONSTRAINT fk_manager
        FOREIGN KEY (manager_id)
        REFERENCES employees(employee_id)
        ON DELETE SET NULL,

    -- Check constraints
    CONSTRAINT chk_hire_after_birth
        CHECK (hire_date > birth_date),

    -- Unique constraint (composite)
    CONSTRAINT uq_name_email
        UNIQUE (first_name, last_name, email)
);
```

#### Common Data Types

```sql
-- Numeric types
INT, INTEGER                 -- Whole numbers (-2B to 2B)
SMALLINT                     -- Small integers (-32K to 32K)
BIGINT                       -- Large integers
DECIMAL(p, s), NUMERIC(p, s) -- Fixed precision (p=total digits, s=decimal places)
FLOAT, REAL                  -- Approximate numbers
DOUBLE PRECISION             -- Double precision float

-- Auto-increment
SERIAL, BIGSERIAL            -- PostgreSQL auto-increment
AUTO_INCREMENT               -- MySQL auto-increment
IDENTITY(1,1)                -- SQL Server auto-increment

-- String types
CHAR(n)                      -- Fixed length (padded with spaces)
VARCHAR(n)                   -- Variable length (up to n characters)
TEXT                         -- Unlimited text (PostgreSQL, MySQL)
NVARCHAR(n)                  -- Unicode variable (SQL Server)

-- Date/Time types
DATE                         -- Date only (YYYY-MM-DD)
TIME                         -- Time only (HH:MM:SS)
DATETIME                     -- Date and time (MySQL, SQL Server)
TIMESTAMP                    -- Date and time with timezone
TIMESTAMPTZ                  -- Timestamp with timezone (PostgreSQL)
INTERVAL                     -- Time interval (PostgreSQL)

-- Boolean
BOOLEAN, BOOL                -- True/false (PostgreSQL)
TINYINT(1)                   -- Boolean in MySQL
BIT                          -- Boolean in SQL Server

-- Binary types
BLOB                         -- Binary large object (MySQL)
BYTEA                        -- Binary data (PostgreSQL)
VARBINARY(n)                 -- Variable binary (SQL Server)

-- Special types
JSON, JSONB                  -- JSON data (JSONB is binary, faster)
XML                          -- XML data
UUID                         -- Universally unique identifier (PostgreSQL)
ENUM('val1', 'val2')         -- Enumerated type (MySQL, PostgreSQL)
ARRAY                        -- Arrays (PostgreSQL: INT[], TEXT[])
```

#### Column Constraint Order

```sql
column_name
    datatype                          -- Required
    [NOT NULL]                        -- Cannot be NULL
    [UNIQUE]                          -- Must be unique
    [PRIMARY KEY]                     -- Primary key (implies NOT NULL + UNIQUE)
    [FOREIGN KEY REFERENCES table(column)]  -- Foreign key reference
    [CHECK (condition)]               -- Custom validation
    [DEFAULT value]                   -- Default value
    [AUTO_INCREMENT]                  -- Auto-increment (MySQL)
```

#### Create Table Variations

```sql
-- Create table from query
CREATE TABLE employee_backup AS
SELECT * FROM employees WHERE hire_date < '2020-01-01';

-- Create temporary table
CREATE TEMPORARY TABLE temp_results (
    id INT,
    value VARCHAR(100)
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id)
);
```

### ALTER TABLE

```sql
-- Add single column
ALTER TABLE employees
ADD COLUMN middle_name VARCHAR(50);

-- Add column with position (MySQL)
ALTER TABLE employees
ADD COLUMN middle_name VARCHAR(50) AFTER first_name;

-- Add multiple columns
ALTER TABLE employees
ADD COLUMN (
    ssn VARCHAR(11),
    tax_id VARCHAR(20)
);

-- Modify column data type
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12, 2);  -- MySQL
-- ALTER COLUMN salary TYPE DECIMAL(12, 2);  -- PostgreSQL

-- Rename column
ALTER TABLE employees
CHANGE old_name new_name VARCHAR(100);  -- MySQL
-- RENAME COLUMN old_name TO new_name;   -- PostgreSQL

-- Drop column
ALTER TABLE employees
DROP COLUMN middle_name;

-- Add constraint
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (email);

ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary >= 0);

-- Drop constraint
ALTER TABLE employees
DROP CONSTRAINT unique_email;

-- Set/Drop default
ALTER TABLE employees
ALTER COLUMN created_at SET DEFAULT NOW();

ALTER TABLE employees
ALTER COLUMN created_at DROP DEFAULT;

-- Set/Drop NOT NULL
ALTER TABLE employees
ALTER COLUMN email SET NOT NULL;

ALTER TABLE employees
ALTER COLUMN email DROP NOT NULL;

-- Rename table
ALTER TABLE old_table_name RENAME TO new_table_name;
```

### DROP, TRUNCATE, RENAME

```sql
-- Drop database
DROP DATABASE database_name;
DROP DATABASE IF EXISTS database_name;

-- Drop table
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables
DROP TABLE IF EXISTS table1, table2, table3;

-- Drop with dependencies (PostgreSQL)
DROP TABLE table_name CASCADE;

-- TRUNCATE - fast delete all rows
TRUNCATE TABLE table_name;

-- TRUNCATE with reset (PostgreSQL)
TRUNCATE TABLE table_name RESTART IDENTITY CASCADE;

-- Rename table
RENAME TABLE old_name TO new_name;  -- MySQL
ALTER TABLE old_name RENAME TO new_name;  -- PostgreSQL
```

### CREATE INDEX

```sql
-- Basic index
CREATE INDEX index_name ON table_name (column_name);

-- Composite index
CREATE INDEX idx_name_email ON employees (last_name, first_name, email);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON employees (email);

-- Partial/filtered index (PostgreSQL)
CREATE INDEX idx_active_employees
ON employees (last_name)
WHERE status = 'active';

-- Full-text index (MySQL)
CREATE FULLTEXT INDEX idx_description ON products (description);

-- Descending index
CREATE INDEX idx_salary_desc ON employees (salary DESC);

-- Expression-based index (PostgreSQL)
CREATE INDEX idx_lower_email ON employees (LOWER(email));

-- Drop index
DROP INDEX index_name ON table_name;  -- MySQL
DROP INDEX index_name;                 -- PostgreSQL
DROP INDEX IF EXISTS index_name;       -- PostgreSQL
```

**Index Best Practices:**
- Index columns used in WHERE, JOIN, ORDER BY
- Index foreign key columns
- Don't index small tables (< 1000 rows)
- Don't index low-cardinality columns (few distinct values)
- Composite index order matters: (country, age) helps "WHERE country = 'US'" but not "WHERE age > 18"

### CREATE VIEW

```sql
-- Basic view
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;

-- Complex view with joins and aggregation
CREATE VIEW employee_details AS
SELECT
    e.employee_id,
    e.first_name,
    e.last_name,
    e.email,
    d.department_name,
    e.salary,
    CASE
        WHEN e.salary > 80000 THEN 'High'
        WHEN e.salary > 50000 THEN 'Medium'
        ELSE 'Low'
    END as salary_bracket
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE e.status = 'active';

-- View with aggregation
CREATE VIEW department_stats AS
SELECT
    d.department_name,
    COUNT(e.employee_id) as employee_count,
    AVG(e.salary) as avg_salary,
    MAX(e.salary) as max_salary,
    MIN(e.salary) as min_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;

-- Updatable view with check option
CREATE VIEW active_employees AS
SELECT employee_id, first_name, last_name, email, status
FROM employees
WHERE status = 'active'
WITH CHECK OPTION;  -- Prevents updates that violate WHERE clause

-- Replace/modify view
CREATE OR REPLACE VIEW view_name AS
SELECT new_columns FROM table_name;

-- Materialized view (PostgreSQL) - stores result physically
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    product_id,
    DATE_TRUNC('month', sale_date) as month,
    SUM(quantity) as total_quantity,
    SUM(amount) as total_amount
FROM sales
GROUP BY product_id, DATE_TRUNC('month', sale_date);

-- Refresh materialized view
REFRESH MATERIALIZED VIEW sales_summary;

-- Drop view
DROP VIEW view_name;
DROP VIEW IF EXISTS view_name;
```

---

## DQL - Data Query Language

### SELECT - Complete Syntax

```sql
SELECT [DISTINCT] column_list          -- 5. Select columns (logical order)
FROM table_name                        -- 1. From tables
[JOIN other_table ON condition]        -- 2. Join tables
[WHERE conditions]                     -- 3. Filter rows
[GROUP BY column_list]                 -- 4. Group rows
[HAVING group_conditions]              -- 6. Filter groups
[ORDER BY column_list [ASC|DESC]]      -- 7. Sort results
[LIMIT count [OFFSET offset]];         -- 8. Limit results
```

### SELECT Basics

```sql
-- Select all columns (avoid in production)
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, email FROM employees;

-- Select with aliases
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary * 12 AS annual_salary,
    'Active' AS status
FROM employees;

-- Select distinct values
SELECT DISTINCT department_id FROM employees;
SELECT DISTINCT country, city FROM customers;  -- Distinct combinations

-- Select with calculations
SELECT
    product_name,
    price,
    quantity,
    price * quantity AS total_value,
    price * quantity * 0.1 AS tax,
    price * quantity * 1.1 AS total_with_tax
FROM products;
```

### WHERE Clause - Filtering

```sql
-- Comparison operators
WHERE salary = 50000              -- Equality
WHERE salary != 50000             -- Not equal (also: <>)
WHERE salary > 50000              -- Greater than
WHERE salary >= 50000             -- Greater or equal
WHERE salary < 50000              -- Less than
WHERE salary <= 50000             -- Less or equal
WHERE salary <> 50000             -- Not equal

-- BETWEEN (inclusive range)
WHERE salary BETWEEN 40000 AND 60000
WHERE hire_date BETWEEN '2020-01-01' AND '2023-12-31'
WHERE age NOT BETWEEN 18 AND 65

-- IN (match any value in list)
WHERE department_id IN (5, 10, 15)
WHERE first_name IN ('John', 'Jane', 'Bob')
WHERE country NOT IN ('US', 'CA')

-- LIKE (pattern matching)
WHERE last_name LIKE 'S%'          -- Starts with S
WHERE last_name LIKE '%son'        -- Ends with son
WHERE last_name LIKE '%mit%'       -- Contains mit
WHERE first_name LIKE 'J___'       -- J followed by exactly 3 chars
WHERE email LIKE '%@gmail.com'     -- Gmail emails
WHERE name LIKE 'John\_Doe'        -- Escape underscore: \_

-- ILIKE (case-insensitive, PostgreSQL)
WHERE last_name ILIKE 's%'

-- REGEXP/RLIKE (MySQL regular expressions)
WHERE email REGEXP '^[a-z]+@company\\.com$'
WHERE phone REGEXP '^[0-9]{3}-[0-9]{3}-[0-9]{4}$'

-- IS NULL / IS NOT NULL
WHERE phone IS NULL
WHERE phone IS NOT NULL
WHERE email IS NULL OR phone IS NULL

-- Logical operators
WHERE salary > 50000 AND department_id = 5
WHERE department_id = 5 OR department_id = 10
WHERE NOT (salary < 30000)
WHERE (status = 'active' OR status = 'pending') AND salary > 40000

-- EXISTS (check if subquery returns any rows)
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = customers.customer_id
)

-- NOT EXISTS
WHERE NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = customers.customer_id
)

-- ANY/SOME (compare to any value in subquery)
WHERE salary > ANY (SELECT salary FROM employees WHERE department_id = 5)
WHERE salary > SOME (SELECT salary FROM employees WHERE department_id = 5)

-- ALL (compare to all values in subquery)
WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 5)
```

### JOIN Operations

```sql
-- INNER JOIN (only matching rows from both tables)
SELECT
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- LEFT JOIN / LEFT OUTER JOIN (all from left, matching from right)
SELECT
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- RIGHT JOIN / RIGHT OUTER JOIN (all from right, matching from left)
SELECT
    e.first_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN (all from both, matching where possible)
SELECT
    e.first_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- CROSS JOIN (Cartesian product - all combinations)
SELECT
    e.first_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;

-- SELF JOIN (join table to itself)
SELECT
    e1.first_name AS employee,
    e2.first_name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;

-- Multiple JOINs
SELECT
    e.first_name,
    e.last_name,
    d.department_name,
    p.project_name,
    l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN employee_projects ep ON e.employee_id = ep.employee_id
INNER JOIN projects p ON ep.project_id = p.project_id
LEFT JOIN locations l ON d.location_id = l.location_id;

-- JOIN with multiple conditions
SELECT *
FROM orders o
INNER JOIN order_items oi
    ON o.order_id = oi.order_id
    AND o.status = 'completed'
    AND oi.quantity > 0;

-- JOIN USING (when column names are the same)
SELECT *
FROM employees e
JOIN departments d USING (department_id);
```

### Aggregate Functions

```sql
-- Basic aggregates
SELECT
    COUNT(*) as total_rows,              -- Count all rows
    COUNT(email) as emails_count,        -- Count non-NULL values
    COUNT(DISTINCT department_id) as dept_count,  -- Count unique values
    SUM(salary) as total_salary,
    AVG(salary) as average_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary,
    MIN(hire_date) as first_hire,
    MAX(hire_date) as last_hire
FROM employees;

-- Statistical aggregates
SELECT
    STDDEV(salary) as salary_std_dev,    -- Standard deviation
    VARIANCE(salary) as salary_variance   -- Variance
FROM employees;

-- String aggregation
SELECT
    department_id,
    GROUP_CONCAT(first_name ORDER BY last_name SEPARATOR ', ') as employees  -- MySQL
    -- STRING_AGG(first_name, ', ' ORDER BY last_name) as employees           -- PostgreSQL
FROM employees
GROUP BY department_id;
```

### GROUP BY and HAVING

```sql
-- Basic GROUP BY
SELECT
    department_id,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department_id;

-- GROUP BY multiple columns
SELECT
    department_id,
    status,
    COUNT(*) as count,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department_id, status;

-- GROUP BY with HAVING (filter groups)
SELECT
    department_id,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5 AND AVG(salary) > 50000;

-- GROUP BY with JOIN
SELECT
    d.department_name,
    COUNT(e.employee_id) as employee_count,
    AVG(e.salary) as avg_salary,
    MAX(e.salary) as max_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
HAVING COUNT(e.employee_id) > 0
ORDER BY employee_count DESC;

-- ROLLUP (subtotals and grand total)
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY department_id, status WITH ROLLUP;  -- MySQL
-- GROUP BY ROLLUP(department_id, status);   -- PostgreSQL

-- CUBE (all possible subtotals)
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY CUBE(department_id, status);  -- PostgreSQL, Oracle, SQL Server

-- GROUPING SETS (custom grouping combinations)
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY GROUPING SETS (
    (department_id, status),  -- Group by both
    (department_id),          -- Group by department only
    (status),                 -- Group by status only
    ()                        -- Grand total
);
```

### ORDER BY

```sql
-- Order by single column
SELECT * FROM employees ORDER BY last_name;
SELECT * FROM employees ORDER BY last_name ASC;   -- Ascending (default)
SELECT * FROM employees ORDER BY salary DESC;     -- Descending

-- Order by multiple columns
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC, last_name ASC;

-- Order by expression
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary * 12 DESC;  -- Annual salary

-- Order by alias
SELECT
    first_name,
    last_name,
    salary * 12 as annual_salary
FROM employees
ORDER BY annual_salary DESC;

-- Order by CASE
SELECT first_name, last_name, department_id
FROM employees
ORDER BY
    CASE department_id
        WHEN 1 THEN 1
        WHEN 5 THEN 2
        ELSE 3
    END,
    last_name;

-- NULL handling (PostgreSQL)
SELECT * FROM employees
ORDER BY phone NULLS FIRST;   -- NULLs appear first
-- ORDER BY phone NULLS LAST;  -- NULLs appear last

-- NULL handling (MySQL - NULLs first in ASC, last in DESC)
SELECT * FROM employees
ORDER BY ISNULL(phone), phone;  -- Force NULLs last in ASC
```

### LIMIT and OFFSET

```sql
-- LIMIT (restrict number of rows)
SELECT * FROM employees LIMIT 10;

-- LIMIT with OFFSET (pagination)
SELECT * FROM employees LIMIT 10 OFFSET 20;  -- Skip 20, return next 10

-- Alternative syntax (MySQL)
SELECT * FROM employees LIMIT 20, 10;  -- Offset 20, limit 10

-- Pagination example
-- Page 1: LIMIT 10 OFFSET 0
-- Page 2: LIMIT 10 OFFSET 10
-- Page 3: LIMIT 10 OFFSET 20

-- SQL Server syntax
SELECT TOP 10 * FROM employees;
SELECT TOP 10 PERCENT * FROM employees;

-- SQL Server with OFFSET (requires ORDER BY)
SELECT * FROM employees
ORDER BY employee_id
OFFSET 20 ROWS
FETCH NEXT 10 ROWS ONLY;

-- Get top N with ties (SQL Server)
SELECT TOP 10 WITH TIES *
FROM employees
ORDER BY salary DESC;
```

### Subqueries

```sql
-- Subquery in WHERE (scalar subquery - returns single value)
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery with IN
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location = 'New York'
);

-- Subquery with EXISTS
SELECT d.department_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);

-- Subquery in SELECT (must return single value per row)
SELECT
    first_name,
    last_name,
    salary,
    (SELECT AVG(salary) FROM employees) as company_avg,
    salary - (SELECT AVG(salary) FROM employees) as diff_from_avg
FROM employees;

-- Subquery in FROM (derived table/inline view)
SELECT
    dept_name,
    avg_salary
FROM (
    SELECT
        d.department_name as dept_name,
        AVG(e.salary) as avg_salary
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    GROUP BY d.department_id, d.department_name
) as dept_stats
WHERE avg_salary > 50000;

-- Correlated subquery (references outer query)
SELECT
    e1.first_name,
    e1.last_name,
    e1.salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);

-- ALL/ANY with subquery
SELECT first_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department_id = 5
);

SELECT first_name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department_id = 5
);
```

### UNION, INTERSECT, EXCEPT

```sql
-- UNION (combine results, remove duplicates)
SELECT first_name, last_name FROM employees
UNION
SELECT first_name, last_name FROM contractors;

-- UNION ALL (combine results, keep duplicates - faster)
SELECT first_name, last_name FROM employees
UNION ALL
SELECT first_name, last_name FROM contractors;

-- Complex UNION with ORDER BY
(
    SELECT
        'Employee' as type,
        first_name,
        last_name,
        email,
        salary
    FROM employees
    WHERE status = 'active'
)
UNION ALL
(
    SELECT
        'Contractor' as type,
        first_name,
        last_name,
        email,
        hourly_rate * 2000 as equivalent_salary
    FROM contractors
    WHERE end_date > CURRENT_DATE
)
ORDER BY type, last_name;

-- INTERSECT (common rows in both queries)
SELECT employee_id FROM employees
INTERSECT
SELECT employee_id FROM project_members;

-- EXCEPT/MINUS (rows in first query but not in second)
SELECT employee_id FROM employees
EXCEPT  -- PostgreSQL, SQL Server
-- MINUS  -- Oracle, MySQL 8.0.31+
SELECT employee_id FROM contractors;
```

### CASE Expressions

```sql
-- Simple CASE
SELECT
    first_name,
    last_name,
    department_id,
    CASE department_id
        WHEN 1 THEN 'Sales'
        WHEN 2 THEN 'Marketing'
        WHEN 3 THEN 'Engineering'
        WHEN 4 THEN 'HR'
        ELSE 'Other'
    END as department_name
FROM employees;

-- Searched CASE
SELECT
    first_name,
    last_name,
    salary,
    CASE
        WHEN salary < 30000 THEN 'Low'
        WHEN salary BETWEEN 30000 AND 60000 THEN 'Medium'
        WHEN salary > 60000 AND salary <= 100000 THEN 'High'
        WHEN salary > 100000 THEN 'Very High'
        ELSE 'Unknown'
    END as salary_bracket
FROM employees;

-- CASE in aggregate
SELECT
    department_id,
    COUNT(*) as total_employees,
    SUM(CASE WHEN salary > 50000 THEN 1 ELSE 0 END) as high_earners,
    SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END) as active_count,
    AVG(CASE WHEN status = 'active' THEN salary END) as avg_active_salary
FROM employees
GROUP BY department_id;

-- CASE in ORDER BY
SELECT first_name, last_name, status
FROM employees
ORDER BY
    CASE status
        WHEN 'active' THEN 1
        WHEN 'inactive' THEN 2
        WHEN 'suspended' THEN 3
        ELSE 4
    END,
    last_name;

-- Nested CASE
SELECT
    first_name,
    department_id,
    salary,
    CASE
        WHEN department_id = 1 THEN
            CASE
                WHEN salary > 50000 THEN 'Sales-High'
                ELSE 'Sales-Standard'
            END
        WHEN department_id = 2 THEN 'Marketing'
        ELSE 'Other'
    END as category
FROM employees;
```

---

## DML - Data Manipulation Language

### INSERT

```sql
-- Insert single row (specify columns - recommended)
INSERT INTO employees (first_name, last_name, email, hire_date, salary)
VALUES ('John', 'Doe', 'john.doe@company.com', '2024-01-15', 55000.00);

-- Insert multiple rows
INSERT INTO employees (first_name, last_name, email, salary) VALUES
    ('Alice', 'Johnson', 'alice@company.com', 52000),
    ('Bob', 'Williams', 'bob@company.com', 58000),
    ('Carol', 'Brown', 'carol@company.com', 61000);

-- Insert with DEFAULT values
INSERT INTO employees (first_name, last_name, email)
VALUES ('Mike', 'Davis', 'mike@company.com');
-- hire_date, created_at, etc. will use DEFAULT values

-- Insert explicit DEFAULT
INSERT INTO employees (first_name, last_name, email, hire_date, status)
VALUES ('Sarah', 'Miller', 'sarah@company.com', DEFAULT, DEFAULT);

-- Insert from SELECT
INSERT INTO employee_backup (employee_id, first_name, last_name, email)
SELECT employee_id, first_name, last_name, email
FROM employees
WHERE hire_date < '2020-01-01';

-- Insert and return values (PostgreSQL)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Tom', 'Wilson', 'tom@company.com')
RETURNING employee_id, first_name, created_at;

-- Insert and get last ID (MySQL)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Lisa', 'Anderson', 'lisa@company.com');
SELECT LAST_INSERT_ID();

-- UPSERT - Insert on duplicate key update (MySQL)
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
VALUES (101, 'John', 'Doe', 'john@company.com', 55000)
ON DUPLICATE KEY UPDATE
    first_name = VALUES(first_name),
    last_name = VALUES(last_name),
    salary = VALUES(salary),
    updated_at = CURRENT_TIMESTAMP;

-- UPSERT - Insert on conflict (PostgreSQL)
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
VALUES (101, 'John', 'Doe', 'john@company.com', 55000)
ON CONFLICT (employee_id)
DO UPDATE SET
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    salary = EXCLUDED.salary,
    updated_at = CURRENT_TIMESTAMP;

-- Insert on conflict do nothing
INSERT INTO employees (email, first_name, last_name)
VALUES ('existing@company.com', 'John', 'Doe')
ON CONFLICT (email) DO NOTHING;

-- Insert with CTE
WITH new_hires AS (
    SELECT * FROM temp_employees
    WHERE status = 'approved'
)
INSERT INTO employees (first_name, last_name, email, department_id)
SELECT first_name, last_name, email, department_id
FROM new_hires;
```

### UPDATE

```sql
-- Update single column
UPDATE employees
SET salary = 60000
WHERE employee_id = 101;

-- Update multiple columns
UPDATE employees
SET
    salary = 65000,
    status = 'active',
    updated_at = CURRENT_TIMESTAMP
WHERE employee_id = 101;

-- Update with expression
UPDATE employees
SET salary = salary * 1.10  -- 10% raise
WHERE department_id = 5;

-- Update with CASE
UPDATE employees
SET salary = CASE
    WHEN department_id = 1 THEN salary * 1.15
    WHEN department_id = 2 THEN salary * 1.10
    WHEN department_id = 3 THEN salary * 1.08
    ELSE salary * 1.05
END
WHERE status = 'active';

-- Update with subquery
UPDATE employees
SET salary = (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = employees.department_id
)
WHERE employee_id = 101;

-- Update with JOIN (MySQL)
UPDATE employees e
INNER JOIN departments d ON e.department_id = d.department_id
SET e.salary = e.salary * 1.10
WHERE d.department_name = 'Sales';

-- Update with FROM (PostgreSQL)
UPDATE employees e
SET salary = e.salary * 1.10
FROM departments d
WHERE e.department_id = d.department_id
AND d.department_name = 'Sales';

-- Update and return (PostgreSQL)
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 5
RETURNING employee_id, first_name, salary;

-- Update with LIMIT (MySQL - limit number of rows updated)
UPDATE employees
SET status = 'inactive'
WHERE last_login < '2020-01-01'
ORDER BY last_login
LIMIT 100;
```

### DELETE

```sql
-- Delete specific rows
DELETE FROM employees
WHERE employee_id = 101;

-- Delete with multiple conditions
DELETE FROM employees
WHERE status = 'inactive'
AND hire_date < '2010-01-01';

-- Delete with subquery
DELETE FROM employees
WHERE employee_id IN (
    SELECT employee_id
    FROM performance_reviews
    WHERE rating < 2
);

-- Delete with JOIN (MySQL)
DELETE e
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Deprecated';

-- Delete with USING (PostgreSQL)
DELETE FROM employees e
USING departments d
WHERE e.department_id = d.department_id
AND d.department_name = 'Deprecated';

-- Delete all rows (use TRUNCATE for better performance)
DELETE FROM temporary_table;

-- Delete with LIMIT (MySQL)
DELETE FROM log_entries
WHERE created_at < '2020-01-01'
ORDER BY created_at
LIMIT 1000;

-- Delete and return (PostgreSQL)
DELETE FROM employees
WHERE status = 'inactive'
RETURNING employee_id, first_name, last_name;

-- Delete with EXISTS
DELETE FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
);
```

### MERGE (UPSERT)

```sql
-- SQL Server MERGE
MERGE INTO employees AS target
USING temp_employees AS source
ON target.employee_id = source.employee_id
WHEN MATCHED THEN
    UPDATE SET
        target.first_name = source.first_name,
        target.last_name = source.last_name,
        target.salary = source.salary,
        target.updated_at = CURRENT_TIMESTAMP
WHEN NOT MATCHED BY TARGET THEN
    INSERT (employee_id, first_name, last_name, email, salary)
    VALUES (source.employee_id, source.first_name, source.last_name,
            source.email, source.salary)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;

-- PostgreSQL equivalent using INSERT ON CONFLICT
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
SELECT employee_id, first_name, last_name, email, salary
FROM temp_employees
ON CONFLICT (employee_id)
DO UPDATE SET
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    salary = EXCLUDED.salary,
    updated_at = CURRENT_TIMESTAMP;
```

---

## DCL - Data Control Language

### GRANT

```sql
-- Grant SELECT permission on a table
GRANT SELECT ON employees TO user_name;

-- Grant multiple permissions
GRANT SELECT, INSERT, UPDATE ON employees TO user_name;

-- Grant all permissions on a table
GRANT ALL PRIVILEGES ON employees TO user_name;

-- Grant permission on all tables in a database (MySQL)
GRANT SELECT ON database_name.* TO user_name;

-- Grant permission on all tables in a schema (PostgreSQL)
GRANT SELECT ON ALL TABLES IN SCHEMA public TO user_name;

-- Grant with GRANT OPTION (allows user to grant to others)
GRANT SELECT ON employees TO user_name WITH GRANT OPTION;

-- Grant specific column permissions
GRANT SELECT (first_name, last_name, email) ON employees TO user_name;
GRANT UPDATE (salary) ON employees TO hr_user;

-- Grant EXECUTE permission on stored procedure/function
GRANT EXECUTE ON PROCEDURE procedure_name TO user_name;
GRANT EXECUTE ON FUNCTION function_name TO user_name;

-- Grant role to user (PostgreSQL, Oracle)
GRANT role_name TO user_name;
```

### REVOKE

```sql
-- Revoke SELECT permission
REVOKE SELECT ON employees FROM user_name;

-- Revoke multiple permissions
REVOKE SELECT, INSERT, UPDATE ON employees FROM user_name;

-- Revoke all permissions
REVOKE ALL PRIVILEGES ON employees FROM user_name;

-- Revoke with CASCADE
REVOKE SELECT ON employees FROM user_name CASCADE;

-- Revoke GRANT OPTION only
REVOKE GRANT OPTION FOR SELECT ON employees FROM user_name;

-- Revoke role from user
REVOKE role_name FROM user_name;
```

### CREATE USER and ROLES

```sql
-- Create user (MySQL)
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
CREATE USER 'username'@'%' IDENTIFIED BY 'password';  -- From any host

-- Create user (PostgreSQL)
CREATE USER username WITH PASSWORD 'password';
CREATE USER username WITH
    PASSWORD 'password'
    VALID UNTIL '2025-12-31'
    CONNECTION LIMIT 10;

-- Create role (PostgreSQL)
CREATE ROLE readonly_role;
CREATE ROLE admin_role WITH LOGIN PASSWORD 'password' CREATEDB;

-- Create role with permissions (PostgreSQL)
CREATE ROLE app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;
GRANT app_readonly TO username;

-- Alter user
ALTER USER username WITH PASSWORD 'new_password';

-- Drop user
DROP USER username;
DROP USER IF EXISTS username;

-- Show grants
SHOW GRANTS FOR username;  -- MySQL
-- \du username             -- PostgreSQL
```

---

## TCL - Transaction Control Language

### Transaction Basics

```sql
-- Begin transaction
BEGIN;                    -- PostgreSQL, MySQL
-- BEGIN TRANSACTION;     -- SQL Server
-- START TRANSACTION;     -- MySQL standard

-- Commit transaction (save changes permanently)
COMMIT;

-- Rollback transaction (undo all changes)
ROLLBACK;

-- Basic transaction example
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;  -- or ROLLBACK if something went wrong
```

### SAVEPOINT

```sql
-- Create savepoint for partial rollback
BEGIN;

INSERT INTO employees (first_name, last_name, email)
VALUES ('John', 'Doe', 'john@company.com');

SAVEPOINT sp1;

UPDATE employees SET salary = 50000 WHERE email = 'john@company.com';

SAVEPOINT sp2;

DELETE FROM employees WHERE email = 'old@company.com';

-- Rollback to savepoint (undoes DELETE only)
ROLLBACK TO SAVEPOINT sp2;

-- Rollback to earlier savepoint (undoes UPDATE and DELETE)
ROLLBACK TO SAVEPOINT sp1;

-- Release savepoint (can't rollback to it anymore)
RELEASE SAVEPOINT sp1;

-- Final commit
COMMIT;
```

### Isolation Levels

```sql
-- Set isolation level (MySQL)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- PostgreSQL syntax
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Check current isolation level
SELECT @@transaction_isolation;  -- MySQL
-- SHOW TRANSACTION ISOLATION LEVEL;  -- PostgreSQL
```

**Isolation Levels Explained:**
- `READ UNCOMMITTED`: Can read uncommitted changes (dirty reads)
- `READ COMMITTED`: Can't read uncommitted changes (default in PostgreSQL)
- `REPEATABLE READ`: Same query returns same results within transaction (default in MySQL)
- `SERIALIZABLE`: Strictest, transactions appear to run sequentially

### Lock Management

```sql
-- Explicit table locks (MySQL)
LOCK TABLES employees WRITE;
-- ... perform operations ...
UNLOCK TABLES;

-- Row-level locks (PostgreSQL)
BEGIN;

-- Lock row for update (prevents other transactions from modifying)
SELECT * FROM employees
WHERE employee_id = 101
FOR UPDATE;

-- Lock row for share (others can read but not update)
SELECT * FROM employees
WHERE employee_id = 101
FOR SHARE;

UPDATE employees SET salary = 60000 WHERE employee_id = 101;

COMMIT;

-- Skip locked rows (PostgreSQL)
SELECT * FROM queue_items
WHERE processed = false
FOR UPDATE SKIP LOCKED
LIMIT 1;

-- Wait for lock or fail immediately (PostgreSQL)
SELECT * FROM employees
WHERE employee_id = 101
FOR UPDATE NOWAIT;
```

---

# Part III: PostgreSQL Specifics

## PostgreSQL Connection & Session

### Connection Methods

```bash
# Basic connection (uses system username)
psql

# Full connection string
psql -U username -d database_name -h localhost -p 5432

# Arguments:
# -U username    User to connect as
# -d database    Database to connect to
# -h hostname    Host to connect to
# -p port        Port number (default: 5432)
# -W             Force password prompt

# Connection string format
psql postgresql://username:password@localhost:5432/database_name

# Execute query and exit
psql -U user -d database -c "SELECT * FROM users LIMIT 5;"

# Execute SQL file
psql -U user -d database -f script.sql

# Execute multiple commands
psql -U user -d database -c "BEGIN;" -c "INSERT INTO..." -c "COMMIT;"
```

## psql Meta-Commands

Meta-commands are psql-specific commands (not SQL) that help navigate and inspect the database.

### Help & Information

```sql
\?                    -- List all psql commands
\h SQL_COMMAND        -- Show SQL syntax help (e.g., \h CREATE TABLE)
\q                    -- Quit psql
\conninfo             -- Show current connection details
```

### Navigation

```sql
\c database_name             -- Connect to different database
\c database_name username    -- Connect as different user
```

### Schema Inspection (Most Important for Development)

```sql
\l                    -- List all databases
\l+                   -- List databases with size and description

\dt                   -- List all tables in current schema
\dt+                  -- List tables with size, description
\dt schema_name.*     -- List tables in specific schema

\d table_name         -- ⭐ MOST IMPORTANT: Show table structure
\d+ table_name        -- Full table details including foreign keys

\dn                   -- List schemas (namespaces)
\dv                   -- List views
\dv+                  -- List views with definition
\di                   -- List indexes
\di+                  -- List indexes with size
\du                   -- List users/roles
\df                   -- List functions
\df+                  -- List functions with definition
\dx                   -- List extensions
\dT                   -- List data types
\ds                   -- List sequences

\d+ *                 -- Show all objects in current schema
```

**Pro Tip**: Always run `\d table_name` before writing queries for that table. It shows exact column names, types, and constraints.

### Display Formatting

```sql
\x on                 -- Expanded display (vertical) - great for wide tables
\x off                -- Table display (horizontal)
\x auto               -- Auto-switch based on width (recommended)

\timing on            -- Show query execution time
\timing off           -- Hide execution time

\pset format wrapped  -- Wrap long lines
\pset format aligned  -- Aligned columns (default)
\pset format unaligned -- Unaligned (for parsing)

\pset null 'NULL'     -- Show NULL values as 'NULL'
```

**Example:**
```sql
\x on
SELECT * FROM employees WHERE id = 1;
-- Output:
-- id         | 1
-- first_name | John
-- last_name  | Doe
-- email      | john@example.com
-- Much easier to read for wide tables!
```

### Output Control

```sql
\o filename.txt       -- Redirect output to file
\o                    -- Stop redirecting, return to screen
\e                    -- Open last query in text editor
\ef function_name     -- Edit function in editor
```

### Data Import/Export

```sql
-- Export table to CSV
\copy employees TO '/path/to/employees.csv' CSV HEADER;

-- Import CSV to table
\copy employees FROM '/path/to/seed.csv' CSV HEADER;

-- Export query results to CSV
\copy (SELECT name, email FROM employees WHERE active) TO '/path/to/active.csv' CSV HEADER;

-- Export with delimiter
\copy employees TO '/path/to/employees.tsv' DELIMITER E'\t' HEADER;
```

**\copy vs COPY**: `\copy` runs on client (can access local files). `COPY` runs on server (needs server file access).

### Monitoring & Debugging

```sql
\watch 2              -- Re-run last query every 2 seconds
                     -- (Great for monitoring changes in real-time)

\echo 'message'       -- Print message (useful in scripts)

\set                  -- Show all variables

\set AUTOCOMMIT off   -- Disable autocommit

\timing on            -- Enable query timing
```

**Example:**
```sql
SELECT count(*) FROM jobs WHERE status = 'running';
\watch 2  -- Updates every 2 seconds
```

---

# Part IV: Procedural SQL

## Variables and Control Flow

### Variables (PostgreSQL - PL/pgSQL)

```sql
DO $$
DECLARE
    -- Variable declarations
    v_employee_id INT := 101;
    v_first_name VARCHAR(50);
    v_last_name VARCHAR(50);
    v_salary NUMERIC(10, 2);
    v_count INT DEFAULT 0;
    v_today DATE := CURRENT_DATE;
    v_is_active BOOLEAN := TRUE;

    -- Record type (like a row)
    v_employee employees%ROWTYPE;

    -- Custom record type
    v_emp_record RECORD;

    -- Array
    v_numbers INT[] := ARRAY[1, 2, 3, 4, 5];

    -- Constant
    c_tax_rate CONSTANT NUMERIC := 0.15;
BEGIN
    -- Assign values
    v_first_name := 'John';
    v_last_name := 'Doe';
    v_salary := 50000.00;

    -- Select into variables
    SELECT first_name, last_name, salary
    INTO v_first_name, v_last_name, v_salary
    FROM employees
    WHERE employee_id = v_employee_id;

    -- Select into record
    SELECT * INTO v_employee
    FROM employees
    WHERE employee_id = v_employee_id;

    -- Use variables
    RAISE NOTICE 'Employee: % %, Salary: %',
        v_first_name, v_last_name, v_salary;

    -- Count rows
    SELECT COUNT(*) INTO v_count
    FROM employees
    WHERE department_id = 5;

    RAISE NOTICE 'Count: %', v_count;
END $$;
```

### Variables (MySQL)

```sql
-- User-defined variables
SET @employee_id = 101;
SET @salary = 50000.00;
SET @dept_name = 'Sales';

-- Multiple assignments
SET @var1 = 1, @var2 = 2, @var3 = 3;

-- Assign from SELECT
SELECT @first_name := first_name,
       @last_name := last_name
FROM employees
WHERE employee_id = @employee_id;

-- Use in queries
SELECT * FROM employees
WHERE salary > @salary;
```

### IF-ELSE Statements

```sql
-- PostgreSQL
DO $$
DECLARE
    v_salary NUMERIC;
    v_bonus NUMERIC;
BEGIN
    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = 101;

    -- Simple IF
    IF v_salary > 60000 THEN
        v_bonus := v_salary * 0.15;
    END IF;

    -- IF-ELSE
    IF v_salary > 60000 THEN
        v_bonus := v_salary * 0.15;
    ELSE
        v_bonus := v_salary * 0.10;
    END IF;

    -- IF-ELSIF-ELSE
    IF v_salary > 80000 THEN
        v_bonus := v_salary * 0.20;
    ELSIF v_salary > 60000 THEN
        v_bonus := v_salary * 0.15;
    ELSIF v_salary > 40000 THEN
        v_bonus := v_salary * 0.10;
    ELSE
        v_bonus := v_salary * 0.05;
    END IF;

    RAISE NOTICE 'Bonus: %', v_bonus;
END $$;

-- SQL Server (T-SQL)
DECLARE @salary DECIMAL(10, 2);
DECLARE @bonus DECIMAL(10, 2);

SELECT @salary = salary
FROM employees
WHERE employee_id = 101;

IF @salary > 80000
    SET @bonus = @salary * 0.20;
ELSE IF @salary > 60000
    SET @bonus = @salary * 0.15;
ELSE
    SET @bonus = @salary * 0.10;

-- IF EXISTS
IF EXISTS (SELECT 1 FROM employees WHERE salary > 100000)
BEGIN
    PRINT 'High earners found';
END
```

### WHILE Loops

```sql
-- PostgreSQL
DO $$
DECLARE
    v_counter INT := 1;
    v_sum INT := 0;
BEGIN
    WHILE v_counter <= 10 LOOP
        v_sum := v_sum + v_counter;
        v_counter := v_counter + 1;

        -- Exit condition
        EXIT WHEN v_sum > 30;

        -- Continue to next iteration
        CONTINUE WHEN v_counter = 5;

        RAISE NOTICE 'Counter: %, Sum: %', v_counter, v_sum;
    END LOOP;

    RAISE NOTICE 'Final sum: %', v_sum;
END $$;

-- SQL Server
DECLARE @counter INT = 1;
DECLARE @sum INT = 0;

WHILE @counter <= 10
BEGIN
    SET @sum = @sum + @counter;
    SET @counter = @counter + 1;

    IF @sum > 30
        BREAK;

    IF @counter = 5
        CONTINUE;

    PRINT 'Counter: ' + CAST(@counter AS VARCHAR);
END
```

### FOR Loops

```sql
-- PostgreSQL FOR loop (integer range)
DO $$
DECLARE
    v_i INT;
    v_sum INT := 0;
BEGIN
    FOR v_i IN 1..10 LOOP
        v_sum := v_sum + v_i;
        RAISE NOTICE 'i: %, sum: %', v_i, v_sum;
    END LOOP;

    -- Reverse FOR loop
    FOR v_i IN REVERSE 10..1 LOOP
        RAISE NOTICE 'Countdown: %', v_i;
    END LOOP;

    -- FOR loop with STEP
    FOR v_i IN 1..10 BY 2 LOOP
        RAISE NOTICE 'Odd number: %', v_i;
    END LOOP;
END $$;

-- PostgreSQL FOR loop (query result)
DO $$
DECLARE
    v_employee RECORD;
BEGIN
    FOR v_employee IN
        SELECT employee_id, first_name, last_name, salary
        FROM employees
        WHERE department_id = 5
        ORDER BY salary DESC
    LOOP
        RAISE NOTICE 'Employee: % %, Salary: %',
            v_employee.first_name,
            v_employee.last_name,
            v_employee.salary;

        -- Update each employee
        UPDATE employees
        SET salary = salary * 1.10
        WHERE employee_id = v_employee.employee_id;
    END LOOP;
END $$;

-- PostgreSQL FOREACH (array iteration)
DO $$
DECLARE
    v_arr INT[] := ARRAY[1, 2, 3, 4, 5];
    v_element INT;
BEGIN
    FOREACH v_element IN ARRAY v_arr LOOP
        RAISE NOTICE 'Element: %', v_element;
    END LOOP;
END $$;
```

---

## Functions and Stored Procedures

### Functions (PostgreSQL)

```sql
-- Basic function
CREATE OR REPLACE FUNCTION calculate_bonus(
    p_employee_id INT
) RETURNS NUMERIC AS $$
DECLARE
    v_salary NUMERIC;
    v_bonus NUMERIC;
BEGIN
    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    IF v_salary > 80000 THEN
        v_bonus := v_salary * 0.20;
    ELSIF v_salary > 60000 THEN
        v_bonus := v_salary * 0.15;
    ELSE
        v_bonus := v_salary * 0.10;
    END IF;

    RETURN v_bonus;
END;
$$ LANGUAGE plpgsql;

-- Call function
SELECT calculate_bonus(101);

-- Function returning table
CREATE OR REPLACE FUNCTION get_high_earners(
    p_min_salary NUMERIC
) RETURNS TABLE (
    employee_id INT,
    full_name VARCHAR,
    salary NUMERIC
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        e.employee_id,
        e.first_name || ' ' || e.last_name,
        e.salary
    FROM employees e
    WHERE e.salary >= p_min_salary
    ORDER BY e.salary DESC;
END;
$$ LANGUAGE plpgsql;

-- Call table function
SELECT * FROM get_high_earners(70000);

-- Function with OUT parameters
CREATE OR REPLACE FUNCTION get_employee_stats(
    p_dept_id INT,
    OUT total_count INT,
    OUT avg_salary NUMERIC,
    OUT max_salary NUMERIC
) AS $$
BEGIN
    SELECT
        COUNT(*),
        AVG(salary),
        MAX(salary)
    INTO total_count, avg_salary, max_salary
    FROM employees
    WHERE department_id = p_dept_id;
END;
$$ LANGUAGE plpgsql;

-- Call OUT parameter function
SELECT * FROM get_employee_stats(5);

-- Drop function
DROP FUNCTION calculate_bonus;
DROP FUNCTION IF EXISTS calculate_bonus;
```

### Functions (MySQL)

```sql
DELIMITER //

CREATE FUNCTION calculate_annual_salary(
    p_employee_id INT
) RETURNS DECIMAL(12, 2)
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE v_salary DECIMAL(10, 2);
    DECLARE v_bonus DECIMAL(10, 2);
    DECLARE v_annual DECIMAL(12, 2);

    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    SET v_bonus = v_salary * 0.10;
    SET v_annual = (v_salary + v_bonus) * 12;

    RETURN v_annual;
END //

DELIMITER ;

-- Call function
SELECT calculate_annual_salary(101);
```

### Stored Procedures (PostgreSQL)

```sql
CREATE OR REPLACE PROCEDURE give_raise(
    p_employee_id INT,
    p_percentage NUMERIC
) AS $$
DECLARE
    v_old_salary NUMERIC;
    v_new_salary NUMERIC;
BEGIN
    -- Get current salary
    SELECT salary INTO v_old_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    -- Calculate new salary
    v_new_salary := v_old_salary * (1 + p_percentage / 100);

    -- Update salary
    UPDATE employees
    SET salary = v_new_salary,
        updated_at = CURRENT_TIMESTAMP
    WHERE employee_id = p_employee_id;

    -- Log the change
    INSERT INTO salary_history (employee_id, old_salary, new_salary, change_date)
    VALUES (p_employee_id, v_old_salary, v_new_salary, CURRENT_TIMESTAMP);

    RAISE NOTICE 'Salary updated from % to %', v_old_salary, v_new_salary;
END;
$$ LANGUAGE plpgsql;

-- Call procedure
CALL give_raise(101, 10);

-- Procedure with INOUT parameters
CREATE OR REPLACE PROCEDURE process_order(
    p_order_id INT,
    INOUT p_status VARCHAR,
    OUT p_total NUMERIC
) AS $$
BEGIN
    SELECT total INTO p_total
    FROM orders
    WHERE order_id = p_order_id;

    p_status := 'PROCESSED';

    UPDATE orders
    SET status = p_status,
        processed_at = CURRENT_TIMESTAMP
    WHERE order_id = p_order_id;
END;
$$ LANGUAGE plpgsql;

-- Drop procedure
DROP PROCEDURE give_raise;
DROP PROCEDURE IF EXISTS give_raise;
```

### Stored Procedures (MySQL)

```sql
DELIMITER //

CREATE PROCEDURE give_raise(
    IN p_employee_id INT,
    IN p_percentage DECIMAL(5, 2),
    OUT p_old_salary DECIMAL(10, 2),
    OUT p_new_salary DECIMAL(10, 2)
)
BEGIN
    -- Get current salary
    SELECT salary INTO p_old_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    -- Calculate new salary
    SET p_new_salary = p_old_salary * (1 + p_percentage / 100);

    -- Update salary
    UPDATE employees
    SET salary = p_new_salary,
        updated_at = CURRENT_TIMESTAMP
    WHERE employee_id = p_employee_id;

    -- Log the change
    INSERT INTO salary_history (employee_id, old_salary, new_salary, change_date)
    VALUES (p_employee_id, p_old_salary, p_new_salary, CURRENT_TIMESTAMP);
END //

DELIMITER ;

-- Call procedure
CALL give_raise(101, 10, @old, @new);
SELECT @old AS old_salary, @new AS new_salary;
```

---

## Triggers

### PostgreSQL Triggers

```sql
-- 1. Create trigger function
CREATE OR REPLACE FUNCTION audit_employee_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO employee_audit (operation, employee_id, changed_at)
        VALUES ('INSERT', NEW.employee_id, CURRENT_TIMESTAMP);
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO employee_audit (operation, employee_id, old_salary, new_salary, changed_at)
        VALUES ('UPDATE', NEW.employee_id, OLD.salary, NEW.salary, CURRENT_TIMESTAMP);
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO employee_audit (operation, employee_id, changed_at)
        VALUES ('DELETE', OLD.employee_id, CURRENT_TIMESTAMP);
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- 2. Create trigger
CREATE TRIGGER employee_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION audit_employee_changes();

-- BEFORE trigger for validation
CREATE OR REPLACE FUNCTION validate_salary()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary < 0 THEN
        RAISE EXCEPTION 'Salary cannot be negative';
    END IF;

    IF NEW.salary > 500000 THEN
        RAISE EXCEPTION 'Salary exceeds maximum allowed';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_salary_trigger
    BEFORE INSERT OR UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION validate_salary();

-- Drop trigger
DROP TRIGGER IF EXISTS employee_audit_trigger ON employees;
```

### MySQL Triggers

```sql
DELIMITER //

-- AFTER INSERT trigger
CREATE TRIGGER employee_after_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (operation, employee_id, new_salary, changed_at)
    VALUES ('INSERT', NEW.employee_id, NEW.salary, NOW());
END //

-- BEFORE INSERT trigger (validation)
CREATE TRIGGER employee_before_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be negative';
    END IF;

    -- Auto-set created_at
    SET NEW.created_at = NOW();
END //

-- BEFORE UPDATE trigger
CREATE TRIGGER employee_before_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Auto-update updated_at
    SET NEW.updated_at = NOW();

    -- Prevent salary decrease
    IF NEW.salary < OLD.salary THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be decreased';
    END IF;
END //

DELIMITER ;

-- Drop trigger
DROP TRIGGER employee_after_insert;
```

---

## Cursors

### PostgreSQL Cursors

```sql
DO $$
DECLARE
    -- Declare cursor
    emp_cursor CURSOR FOR
        SELECT employee_id, first_name, last_name, salary
        FROM employees
        WHERE department_id = 5;

    v_emp_id INT;
    v_first_name VARCHAR(50);
    v_last_name VARCHAR(50);
    v_salary NUMERIC;
BEGIN
    -- Open cursor
    OPEN emp_cursor;

    -- Fetch loop
    LOOP
        FETCH emp_cursor INTO v_emp_id, v_first_name, v_last_name, v_salary;

        -- Exit when no more rows
        EXIT WHEN NOT FOUND;

        -- Process row
        RAISE NOTICE 'Employee: % % (%), Salary: %',
            v_first_name, v_last_name, v_emp_id, v_salary;

        -- Update employee
        UPDATE employees
        SET salary = salary * 1.05
        WHERE employee_id = v_emp_id;
    END LOOP;

    -- Close cursor
    CLOSE emp_cursor;
END $$;

-- Simpler cursor with FOR loop
DO $$
DECLARE
    emp_cursor CURSOR FOR
        SELECT employee_id, first_name, salary
        FROM employees
        WHERE department_id = 5;
    v_record RECORD;
BEGIN
    FOR v_record IN emp_cursor LOOP
        RAISE NOTICE 'Employee: %, Salary: %',
            v_record.first_name, v_record.salary;
    END LOOP;
END $$;
```

### MySQL Cursors

```sql
DELIMITER //

CREATE PROCEDURE process_employees()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_emp_id INT;
    DECLARE v_first_name VARCHAR(50);
    DECLARE v_salary DECIMAL(10, 2);

    -- Declare cursor
    DECLARE emp_cursor CURSOR FOR
        SELECT employee_id, first_name, salary
        FROM employees
        WHERE department_id = 5;

    -- Declare handler for end of cursor
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- Open cursor
    OPEN emp_cursor;

    -- Fetch loop
    read_loop: LOOP
        FETCH emp_cursor INTO v_emp_id, v_first_name, v_salary;

        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process row
        SELECT CONCAT('Employee: ', v_first_name, ', Salary: ', v_salary);

        UPDATE employees
        SET salary = salary * 1.05
        WHERE employee_id = v_emp_id;
    END LOOP;

    -- Close cursor
    CLOSE emp_cursor;
END //

DELIMITER ;
```

---

## Exception Handling

### PostgreSQL Exception Handling

```sql
DO $$
DECLARE
    v_employee_id INT := 999;
    v_salary NUMERIC;
BEGIN
    -- Try to get salary
    SELECT salary INTO STRICT v_salary
    FROM employees
    WHERE employee_id = v_employee_id;

    RAISE NOTICE 'Salary: %', v_salary;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE NOTICE 'Employee not found';
    WHEN TOO_MANY_ROWS THEN
        RAISE NOTICE 'Multiple employees found';
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
        RAISE NOTICE 'Error code: %', SQLSTATE;
END $$;

-- Custom exception
DO $$
DECLARE
    v_salary NUMERIC := -1000;
BEGIN
    IF v_salary < 0 THEN
        RAISE EXCEPTION 'Invalid salary: %', v_salary
            USING HINT = 'Salary must be positive';
    END IF;

EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error occurred: %', SQLERRM;
        -- Re-raise the exception
        RAISE;
END $$;

-- Nested exception blocks
CREATE OR REPLACE PROCEDURE process_employee(p_emp_id INT)
AS $$
BEGIN
    -- Outer block
    BEGIN
        UPDATE employees
        SET salary = salary * 1.10
        WHERE employee_id = p_emp_id;

        -- Inner block
        BEGIN
            INSERT INTO audit_log (message)
            VALUES ('Salary updated for employee ' || p_emp_id);
        EXCEPTION
            WHEN OTHERS THEN
                RAISE NOTICE 'Audit logging failed, continuing...';
        END;

    EXCEPTION
        WHEN OTHERS THEN
            RAISE NOTICE 'Update failed: %', SQLERRM;
            RAISE;  -- Re-raise exception
    END;
END;
$$ LANGUAGE plpgsql;
```

### MySQL Exception Handling

```sql
DELIMITER //

CREATE PROCEDURE transfer_funds(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL(10, 2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        -- Rollback on error
        ROLLBACK;
        SELECT 'Transaction failed' AS message;
    END;

    START TRANSACTION;

    -- Deduct from source
    UPDATE accounts
    SET balance = balance - amount
    WHERE account_id = from_account;

    -- Add to destination
    UPDATE accounts
    SET balance = balance + amount
    WHERE account_id = to_account;

    COMMIT;
    SELECT 'Transaction successful' AS message;
END //

-- Specific error handlers
CREATE PROCEDURE insert_employee(
    IN p_email VARCHAR(100),
    IN p_first_name VARCHAR(50),
    IN p_last_name VARCHAR(50)
)
BEGIN
    -- Handler for duplicate key
    DECLARE CONTINUE HANDLER FOR 1062
    BEGIN
        SELECT 'Duplicate email address' AS error_message;
    END;

    -- Handler for any SQL exception
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1
            @sqlstate = RETURNED_SQLSTATE,
            @errno = MYSQL_ERRNO,
            @text = MESSAGE_TEXT;
        SELECT @sqlstate, @errno, @text;
        ROLLBACK;
    END;

    START TRANSACTION;

    INSERT INTO employees (email, first_name, last_name)
    VALUES (p_email, p_first_name, p_last_name);

    COMMIT;
END //

DELIMITER ;
```

---

# Part V: Advanced Features

## Window Functions

```sql
-- ROW_NUMBER (unique sequential number)
SELECT
    first_name,
    last_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as row_num
FROM employees;

-- RANK (same rank for ties, skips next rank)
SELECT
    first_name,
    last_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;
-- Result: 1, 2, 2, 4, 5 (if two people tied for 2nd)

-- DENSE_RANK (same rank for ties, no gaps)
SELECT
    first_name,
    last_name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;
-- Result: 1, 2, 2, 3, 4 (if two people tied for 2nd)

-- PARTITION BY (reset for each group)
SELECT
    first_name,
    last_name,
    department_id,
    salary,
    RANK() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) as dept_rank
FROM employees;

-- NTILE (divide into N buckets)
SELECT
    first_name,
    last_name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) as quartile
FROM employees;

-- LAG (access previous row)
SELECT
    employee_id,
    first_name,
    salary,
    LAG(salary, 1) OVER (ORDER BY employee_id) as prev_salary,
    salary - LAG(salary, 1) OVER (ORDER BY employee_id) as salary_diff
FROM employees;

-- LEAD (access next row)
SELECT
    employee_id,
    first_name,
    salary,
    LEAD(salary, 1) OVER (ORDER BY employee_id) as next_salary
FROM employees;

-- FIRST_VALUE and LAST_VALUE
SELECT
    first_name,
    department_id,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) as highest_in_dept,
    LAST_VALUE(salary) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as lowest_in_dept
FROM employees;

-- Aggregate window functions
SELECT
    first_name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) as dept_avg,
    SUM(salary) OVER (PARTITION BY department_id) as dept_total,
    COUNT(*) OVER (PARTITION BY department_id) as dept_count
FROM employees;

-- Running total
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM orders;

-- Moving average (last 3 rows)
SELECT
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3
FROM orders;
```

---

## Common Table Expressions (CTEs)

```sql
-- Basic CTE
WITH employee_avg AS (
    SELECT
        department_id,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT
    e.first_name,
    e.last_name,
    e.salary,
    ea.avg_salary
FROM employees e
JOIN employee_avg ea ON e.department_id = ea.department_id
WHERE e.salary > ea.avg_salary;

-- Multiple CTEs
WITH
dept_stats AS (
    SELECT
        department_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
),
high_salary_depts AS (
    SELECT department_id
    FROM dept_stats
    WHERE avg_salary > 60000
)
SELECT
    e.first_name,
    e.last_name,
    e.salary
FROM employees e
JOIN high_salary_depts hsd ON e.department_id = hsd.department_id;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member (top-level employees)
    SELECT
        employee_id,
        first_name,
        last_name,
        manager_id,
        1 as level,
        CAST(first_name AS VARCHAR(1000)) as path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        eh.level + 1,
        CONCAT(eh.path, ' > ', e.first_name)
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT
    REPEAT('  ', level - 1) || first_name || ' ' || last_name as employee,
    level,
    path
FROM employee_hierarchy
ORDER BY path;

-- Recursive CTE for category tree with totals
WITH RECURSIVE category_totals AS (
    -- Leaf categories
    SELECT
        category_id,
        parent_id,
        name,
        revenue,
        revenue AS total_revenue
    FROM categories
    WHERE category_id NOT IN (
        SELECT DISTINCT parent_id
        FROM categories
        WHERE parent_id IS NOT NULL
    )

    UNION ALL

    -- Parent categories
    SELECT
        c.category_id,
        c.parent_id,
        c.name,
        c.revenue,
        c.revenue + COALESCE(SUM(ct.total_revenue), 0)
    FROM categories c
    LEFT JOIN category_totals ct ON ct.parent_id = c.category_id
    GROUP BY c.category_id, c.parent_id, c.name, c.revenue
)
SELECT * FROM category_totals
ORDER BY category_id;
```

---

## JSON Operations

### PostgreSQL JSON

```sql
-- Create table with JSON column
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB  -- JSONB is binary, more efficient than JSON
);

-- Insert JSON data
INSERT INTO products (name, attributes) VALUES
    ('Laptop', '{"brand": "Dell", "ram": 16, "storage": 512}'),
    ('Phone', '{"brand": "Apple", "model": "iPhone 15", "storage": 256}');

-- Query JSON fields
SELECT
    name,
    attributes->>'brand' AS brand,
    attributes->>'storage' AS storage
FROM products;

-- Query nested JSON
SELECT
    name,
    attributes->'specs'->>'cpu' AS cpu
FROM products
WHERE attributes->>'brand' = 'Dell';

-- JSON array operations
SELECT
    name,
    jsonb_array_elements(attributes->'colors') AS color
FROM products;

-- Update JSON field
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{storage}',
    '1024'
)
WHERE id = 1;

-- Add JSON field
UPDATE products
SET attributes = attributes || '{"warranty": "2 years"}'::jsonb
WHERE id = 1;

-- Check if JSON key exists
SELECT * FROM products
WHERE attributes ? 'warranty';

-- Check if any key in array exists
SELECT * FROM products
WHERE attributes ?| array['brand', 'model'];

-- Check if all keys in array exist
SELECT * FROM products
WHERE attributes ?& array['brand', 'storage'];
```

### MySQL JSON

```sql
-- Create table with JSON column
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    attributes JSON
);

-- Insert JSON
INSERT INTO products (name, attributes)
VALUES ('Laptop', '{"brand": "Dell", "ram": 16, "storage": 512}');

-- Query JSON
SELECT
    name,
    JSON_EXTRACT(attributes, '$.brand') AS brand,
    attributes->>'$.storage' AS storage
FROM products;

-- Update JSON
UPDATE products
SET attributes = JSON_SET(
    attributes,
    '$.storage', 1024,
    '$.warranty', '2 years'
)
WHERE id = 1;

-- JSON_CONTAINS
SELECT * FROM products
WHERE JSON_CONTAINS(attributes, '"Dell"', '$.brand');
```

---

## Full-Text Search

### MySQL Full-Text Search

```sql
-- Create table with FULLTEXT index
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    FULLTEXT KEY ft_title_content (title, content)
);

-- Natural language search
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('database performance');

-- Boolean mode search
SELECT * FROM articles
WHERE MATCH(title, content)
AGAINST('+database -mysql' IN BOOLEAN MODE);

-- Search operators:
-- +     : must include
-- -     : must not include
-- *     : wildcard
-- " "   : exact phrase
-- > <   : increase/decrease relevance

-- Query expansion
SELECT * FROM articles
WHERE MATCH(title, content)
AGAINST('database' WITH QUERY EXPANSION);
```

### PostgreSQL Full-Text Search

```sql
-- Simple text search
SELECT * FROM articles
WHERE to_tsvector('english', title || ' ' || content)
    @@ to_tsquery('english', 'database & performance');

-- Create GIN index for performance
CREATE INDEX idx_article_fts
ON articles
USING GIN (to_tsvector('english', title || ' ' || content));

-- Search with ranking
SELECT
    title,
    ts_rank(to_tsvector('english', content), query) AS rank
FROM articles, to_tsquery('english', 'database & performance') query
WHERE to_tsvector('english', content) @@ query
ORDER BY rank DESC;

-- Add tsvector column for better performance
ALTER TABLE articles ADD COLUMN search_vector tsvector;

UPDATE articles
SET search_vector =
    to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''));

CREATE INDEX idx_search_vector ON articles USING GIN(search_vector);

-- Auto-update trigger
CREATE TRIGGER articles_search_update
BEFORE INSERT OR UPDATE ON articles
FOR EACH ROW EXECUTE FUNCTION
tsvector_update_trigger(search_vector, 'pg_catalog.english', title, content);
```

---

## Partitioning

### MySQL Partitioning

```sql
-- RANGE partitioning
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- LIST partitioning
CREATE TABLE employees_partitioned (
    id INT,
    name VARCHAR(50),
    department_id INT
)
PARTITION BY LIST (department_id) (
    PARTITION p_sales VALUES IN (1, 2, 3),
    PARTITION p_engineering VALUES IN (4, 5, 6),
    PARTITION p_hr VALUES IN (7, 8)
);

-- HASH partitioning
CREATE TABLE customers (
    id INT,
    name VARCHAR(100)
)
PARTITION BY HASH (id)
PARTITIONS 4;
```

### PostgreSQL Declarative Partitioning

```sql
-- Range partitioning
CREATE TABLE sales (
    id SERIAL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2023_q1 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');

CREATE TABLE sales_2023_q2 PARTITION OF sales
    FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');

-- List partitioning
CREATE TABLE employees (
    id SERIAL,
    name VARCHAR(50),
    region VARCHAR(50)
) PARTITION BY LIST (region);

CREATE TABLE employees_north PARTITION OF employees
    FOR VALUES IN ('North', 'Northeast');

CREATE TABLE employees_south PARTITION OF employees
    FOR VALUES IN ('South', 'Southeast');

-- Hash partitioning
CREATE TABLE measurements (
    id SERIAL,
    sensor_id INT,
    value NUMERIC
) PARTITION BY HASH (sensor_id);

CREATE TABLE measurements_p1 PARTITION OF measurements
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE measurements_p2 PARTITION OF measurements
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
```

---

## Indexes and Performance

### Index Types

```sql
-- B-tree index (default, most common)
CREATE INDEX idx_email ON employees (email);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON employees (email);

-- Composite index (order matters!)
CREATE INDEX idx_name_dept ON employees (last_name, first_name, department_id);
-- Helps: WHERE last_name = 'Smith'
-- Helps: WHERE last_name = 'Smith' AND first_name = 'John'
-- Doesn't help: WHERE first_name = 'John' (not leftmost column)

-- Partial index (PostgreSQL - index subset of rows)
CREATE INDEX idx_active_employees
ON employees (last_name)
WHERE status = 'active';

-- Expression-based index (PostgreSQL)
CREATE INDEX idx_lower_email ON employees (LOWER(email));

-- Full-text index (MySQL)
CREATE FULLTEXT INDEX idx_description ON products (description);

-- GIN index (PostgreSQL - for arrays, JSONB, full-text search)
CREATE INDEX idx_tags ON articles USING GIN(tags);
CREATE INDEX idx_jsonb ON products USING GIN(attributes);

-- GiST index (PostgreSQL - for geometric data, ranges)
CREATE INDEX idx_location ON stores USING GIST(location);
```

### Performance Analysis

```sql
-- EXPLAIN - show query plan
EXPLAIN SELECT * FROM employees WHERE email = 'test@example.com';

-- EXPLAIN ANALYZE - run query and show actual timing
EXPLAIN ANALYZE
SELECT * FROM employees WHERE email = 'test@example.com';

-- PostgreSQL - detailed output
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT * FROM employees WHERE salary > 50000;

-- Look for in EXPLAIN output:
-- - Seq Scan (bad for large tables) vs Index Scan (good)
-- - Cost estimates
-- - Actual time vs estimated time
-- - Rows processed
```

### Index Best Practices

```sql
-- ✅ DO: Index foreign key columns
CREATE INDEX idx_dept_id ON employees(department_id);

-- ✅ DO: Index columns used in WHERE, JOIN, ORDER BY
CREATE INDEX idx_hire_date ON employees(hire_date);
CREATE INDEX idx_salary ON employees(salary);

-- ✅ DO: Use composite indexes for multiple columns frequently queried together
CREATE INDEX idx_status_dept ON employees(status, department_id);

-- ❌ DON'T: Index small tables (< 1000 rows)
-- The overhead isn't worth it

-- ❌ DON'T: Index low-cardinality columns (few distinct values)
-- e.g., boolean, status with only 2-3 values
-- Exception: Use partial index for common values

-- ❌ DON'T: Index every column
-- Indexes slow down INSERT/UPDATE/DELETE

-- ✅ DO: Drop unused indexes
DROP INDEX idx_unused;

-- ✅ DO: Monitor index usage (PostgreSQL)
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0
AND indexname NOT LIKE '%_pkey';
```

---

# Part VI: Best Practices

## Common Patterns

### Pattern 1: Pagination

```sql
-- Basic pagination
SELECT * FROM products
ORDER BY product_id
LIMIT 20 OFFSET 0;  -- Page 1

SELECT * FROM products
ORDER BY product_id
LIMIT 20 OFFSET 20;  -- Page 2

-- Better: Keyset pagination (faster for large tables)
-- Page 1
SELECT * FROM products
ORDER BY product_id
LIMIT 20;

-- Page 2 (where last_id is the last ID from page 1)
SELECT * FROM products
WHERE product_id > :last_id
ORDER BY product_id
LIMIT 20;
```

### Pattern 2: Finding Duplicates

```sql
-- Find duplicate emails
SELECT
    email,
    COUNT(*) as count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;

-- Get all rows that have duplicates
SELECT e1.*
FROM employees e1
INNER JOIN (
    SELECT email
    FROM employees
    GROUP BY email
    HAVING COUNT(*) > 1
) e2 ON e1.email = e2.email;

-- Delete duplicates (keep oldest)
DELETE e1
FROM employees e1
INNER JOIN employees e2
    ON e1.email = e2.email
    AND e1.created_at > e2.created_at;
```

### Pattern 3: Upsert (Insert or Update)

```sql
-- PostgreSQL
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
VALUES (101, 'John', 'Doe', 'john@company.com', 55000)
ON CONFLICT (employee_id)
DO UPDATE SET
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    salary = EXCLUDED.salary,
    updated_at = CURRENT_TIMESTAMP;

-- MySQL
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
VALUES (101, 'John', 'Doe', 'john@company.com', 55000)
ON DUPLICATE KEY UPDATE
    first_name = VALUES(first_name),
    last_name = VALUES(last_name),
    salary = VALUES(salary),
    updated_at = CURRENT_TIMESTAMP;
```

### Pattern 4: Conditional Aggregation

```sql
-- Count employees by status using CASE
SELECT
    department_id,
    COUNT(*) as total_employees,
    SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END) as active_count,
    SUM(CASE WHEN status = 'inactive' THEN 1 ELSE 0 END) as inactive_count,
    SUM(CASE WHEN salary > 50000 THEN 1 ELSE 0 END) as high_earners
FROM employees
GROUP BY department_id;
```

### Pattern 5: Running Totals

```sql
-- Running total with window function
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) as running_total
FROM orders;

-- Running total per customer
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) as customer_running_total
FROM orders;
```

### Pattern 6: Pivot Data

```sql
-- Convert rows to columns using CASE
SELECT
    employee_id,
    MAX(CASE WHEN skill_name = 'SQL' THEN proficiency END) AS sql_skill,
    MAX(CASE WHEN skill_name = 'Python' THEN proficiency END) AS python_skill,
    MAX(CASE WHEN skill_name = 'Java' THEN proficiency END) AS java_skill
FROM employee_skills
GROUP BY employee_id;
```

### Pattern 7: Safe Production Updates

```sql
-- Always use transactions for manual updates
BEGIN;

UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 5;

-- Verify changes before committing
SELECT department_id, COUNT(*), AVG(salary)
FROM employees
WHERE department_id = 5
GROUP BY department_id;

-- If correct: COMMIT
-- If wrong: ROLLBACK
COMMIT;
```

### Pattern 8: Debugging Orphaned Records

```sql
-- Find orders with no customer (orphaned records)
SELECT o.*
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;

-- Find customers with no orders
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: SELECT * in Production

```sql
-- ❌ BAD
SELECT * FROM employees;

-- ✅ GOOD
SELECT employee_id, first_name, last_name, email, salary
FROM employees;
```

**Why:**
- Wastes bandwidth retrieving unnecessary columns
- Breaks if columns are added/removed
- Slower performance
- Unclear what data you actually need

### Anti-Pattern 2: Missing WHERE in UPDATE/DELETE

```sql
-- ❌ DISASTER - Updates EVERY row!
UPDATE employees
SET role = 'admin';

-- ✅ CORRECT
UPDATE employees
SET role = 'admin'
WHERE employee_id = 1;

-- ❌ DISASTER - Deletes EVERYTHING!
DELETE FROM employees;

-- ✅ CORRECT
DELETE FROM employees
WHERE status = 'inactive' AND hire_date < '2010-01-01';
```

### Anti-Pattern 3: N+1 Queries

```sql
-- ❌ BAD: Query in a loop (if you're doing this in application code)
-- First query: SELECT * FROM customers;
-- Then for each customer: SELECT * FROM orders WHERE customer_id = ?
-- Results in 1 + N queries!

-- ✅ GOOD: Single query with JOIN
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    o.total
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

### Anti-Pattern 4: Not Using Indexes

```sql
-- ❌ SLOW on large tables without index
SELECT * FROM employees WHERE email = 'test@example.com';

-- ✅ ADD INDEX
CREATE INDEX idx_email ON employees(email);

-- ✅ NOW FAST
SELECT * FROM employees WHERE email = 'test@example.com';
```

### Anti-Pattern 5: Using LIKE with Leading Wildcard

```sql
-- ❌ BAD: Cannot use index (full table scan)
SELECT * FROM employees WHERE last_name LIKE '%son';

-- ✅ BETTER: Can use index
SELECT * FROM employees WHERE last_name LIKE 'John%';

-- ✅ BEST: Exact match uses index
SELECT * FROM employees WHERE last_name = 'Johnson';
```

### Anti-Pattern 6: Not Using Transactions

```sql
-- ❌ BAD: If second UPDATE fails, money disappears!
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- ✅ GOOD: Both succeed or both fail
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```

### Anti-Pattern 7: Premature Optimization

```sql
-- ❌ DON'T: Index every column "just in case"
CREATE INDEX idx_first_name ON employees(first_name);
CREATE INDEX idx_last_name ON employees(last_name);
CREATE INDEX idx_email ON employees(email);
CREATE INDEX idx_phone ON employees(phone);
-- ... every column indexed!

-- ✅ DO: Index based on actual queries
-- First, identify slow queries using EXPLAIN
-- Then, add indexes for columns actually used in WHERE, JOIN, ORDER BY
```

### Anti-Pattern 8: Storing Calculated Values

```sql
-- ❌ BAD: Storing calculated age (gets outdated)
CREATE TABLE users (
    id INT,
    birth_date DATE,
    age INT  -- Will be wrong next year!
);

-- ✅ GOOD: Calculate on the fly
CREATE TABLE users (
    id INT,
    birth_date DATE
);

SELECT
    id,
    birth_date,
    DATE_PART('year', AGE(birth_date)) as age
FROM users;

-- ✅ OR: Use generated column (PostgreSQL 12+, MySQL 5.7+)
CREATE TABLE users (
    id INT,
    birth_date DATE,
    age INT GENERATED ALWAYS AS (DATE_PART('year', AGE(birth_date))) STORED
);
```

---

## Database Migration Best Practices

### Migration File Naming

```
migrations/
├── 001_create_users_table.sql
├── 002_create_departments_table.sql
├── 003_add_users_department_fk.sql
├── 004_add_email_index.sql
├── 005_seed_default_departments.sql
└── 006_create_orders_table.sql
```

### Migration Principles

1. **Incremental**: Each migration is a small, atomic change
2. **Reversible**: Create both UP and DOWN migrations
3. **Idempotent**: Use `IF NOT EXISTS` / `IF EXISTS`
4. **Sequential**: Number migrations, run in order
5. **Tested**: Test both UP and DOWN migrations

### Migration Template

```sql
-- ==========================================
-- Migration: 001_create_users_table
-- Description: Create users table with basic fields
-- Author: Your Name
-- Date: 2024-01-15
-- ==========================================

-- UP Migration
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);

-- Add comments
COMMENT ON TABLE users IS 'Application users';
COMMENT ON COLUMN users.password_hash IS 'Bcrypt hash of user password';

-- ==========================================
-- DOWN Migration (001_create_users_table_down.sql)
-- ==========================================

DROP TABLE IF EXISTS users CASCADE;
```

### Complex Migration Example

```sql
-- ==========================================
-- Migration: 010_add_user_roles
-- ==========================================

-- UP Migration
BEGIN;

-- 1. Create roles table
CREATE TABLE IF NOT EXISTS roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Create junction table
CREATE TABLE IF NOT EXISTS user_roles (
    user_id INT NOT NULL,
    role_id INT NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

-- 3. Add indexes
CREATE INDEX IF NOT EXISTS idx_user_roles_user ON user_roles(user_id);
CREATE INDEX IF NOT EXISTS idx_user_roles_role ON user_roles(role_id);

-- 4. Seed default roles
INSERT INTO roles (name, description) VALUES
    ('admin', 'Administrator with full access'),
    ('user', 'Regular user with limited access'),
    ('moderator', 'Moderator with content management access')
ON CONFLICT (name) DO NOTHING;

COMMIT;

-- ==========================================
-- DOWN Migration
-- ==========================================

BEGIN;

DROP TABLE IF EXISTS user_roles CASCADE;
DROP TABLE IF EXISTS roles CASCADE;

COMMIT;
```

### Migration Checklist

- [ ] Migration is numbered sequentially
- [ ] Both UP and DOWN migrations exist
- [ ] Uses `IF NOT EXISTS` / `IF EXISTS` for idempotency
- [ ] Wrapped in transaction (if database supports DDL in transactions)
- [ ] Foreign keys created after all parent tables
- [ ] Indexes added for foreign keys and commonly queried columns
- [ ] Tested on development database
- [ ] Tested rollback (DOWN migration)
- [ ] Documented with comments
- [ ] No destructive changes without backup

---

## Quick Reference Tables

### Data Types Comparison

| Purpose | PostgreSQL | MySQL | SQL Server |
|---------|-----------|-------|------------|
| Auto-increment | SERIAL | AUTO_INCREMENT | IDENTITY(1,1) |
| Text (unlimited) | TEXT | TEXT | VARCHAR(MAX) |
| Boolean | BOOLEAN | TINYINT(1) | BIT |
| Timestamp with TZ | TIMESTAMPTZ | TIMESTAMP | DATETIMEOFFSET |
| JSON | JSONB | JSON | NVARCHAR(MAX) |
| UUID | UUID | CHAR(36) | UNIQUEIDENTIFIER |

### Operator Differences

| Operation | PostgreSQL | MySQL | SQL Server |
|-----------|-----------|-------|------------|
| String concatenation | `\|\|` or CONCAT() | CONCAT() | + or CONCAT() |
| Case-insensitive LIKE | ILIKE | LIKE | LIKE |
| Regex match | ~ | REGEXP | LIKE or PATINDEX |
| Not equal | <> or != | <> or != | <> or != |
| Date difference | AGE() | DATEDIFF() | DATEDIFF() |

### Function Differences

| Purpose | PostgreSQL | MySQL | SQL Server |
|---------|-----------|-------|------------|
| Current timestamp | CURRENT_TIMESTAMP | NOW() | GETDATE() |
| String length | LENGTH() | LENGTH() | LEN() |
| Substring | SUBSTRING() | SUBSTRING() | SUBSTRING() |
| Current user | CURRENT_USER | USER() | CURRENT_USER |
| Last insert ID | RETURNING | LAST_INSERT_ID() | SCOPE_IDENTITY() |

---

## Summary - Key Concepts

### 1. Execution Order

**Script Level**: DDL → DML → DQL → DCL → TCL

**Statement Level (SELECT)**:
- **Lexical** (write): SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
- **Logical** (execute): FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT

### 2. Golden Rules

- Build structure (DDL) before adding data (DML)
- Parent tables before child tables
- Create objects before granting permissions
- Aliases unavailable in WHERE, available in ORDER BY
- Use transactions for related operations

### 3. Performance Best Practices

- Index foreign keys, WHERE/JOIN columns
- Use EXPLAIN to analyze queries
- Avoid SELECT * in production
- Use appropriate data types
- Normalize to reduce redundancy, denormalize for read performance

### 4. Safety Best Practices

- Always use WHERE in UPDATE/DELETE
- Test queries with LIMIT first
- Wrap manual changes in transactions
- Use IF NOT EXISTS / IF EXISTS
- Back up before destructive operations

### 5. Code Organization

- Use clear table/column names
- Add comments and documentation
- Follow consistent naming conventions
- Version control your schema
- Create migration scripts

---

**Last Updated**: 2026-01-02

This comprehensive reference covers everything from basic SQL to advanced features. Bookmark for quick reference during SQL development!
