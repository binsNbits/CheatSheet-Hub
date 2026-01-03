# Complete SQL Cheatsheet (DDL, DQL, DML, DCL, TCL)

A comprehensive reference for all SQL commands, procedural programming, and advanced features.

---

## Table of Contents
1. [DDL - Data Definition Language](#ddl---data-definition-language)
2. [DQL - Data Query Language](#dql---data-query-language)
3. [DML - Data Manipulation Language](#dml---data-manipulation-language)
4. [DCL - Data Control Language](#dcl---data-control-language)
5. [TCL - Transaction Control Language](#tcl---transaction-control-language)
6. [Procedural SQL](#procedural-sql)
7. [Advanced Features](#advanced-features)

---

## DDL - Data Definition Language

Commands that define and modify database structure and schema.

### CREATE DATABASE

```sql
-- Basic syntax
CREATE DATABASE database_name;

-- With character set and collation
CREATE DATABASE database_name
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- PostgreSQL with encoding
CREATE DATABASE database_name
    ENCODING 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8';

-- If not exists
CREATE DATABASE IF NOT EXISTS database_name;
```

### CREATE TABLE

```sql
-- Basic syntax
CREATE TABLE table_name (
    column1 datatype [constraints],
    column2 datatype [constraints],
    ...
    [table_constraints]
);

-- Comprehensive example
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

    -- Foreign key
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

-- Create table from query
CREATE TABLE employee_backup AS
SELECT * FROM employees WHERE hire_date < '2020-01-01';

-- Create temporary table
CREATE TEMPORARY TABLE temp_results (
    id INT,
    value VARCHAR(100)
);

-- Create table with partitioning (MySQL)
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023)
);
```

### ALTER TABLE

```sql
-- Add column
ALTER TABLE table_name
ADD column_name datatype [constraints];

ALTER TABLE employees
ADD middle_name VARCHAR(50) AFTER first_name;  -- MySQL
-- ADD COLUMN middle_name VARCHAR(50);         -- PostgreSQL

-- Add multiple columns
ALTER TABLE employees
ADD (
    ssn VARCHAR(11),
    tax_id VARCHAR(20)
);

-- Modify column
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12, 2);  -- MySQL
-- ALTER COLUMN salary TYPE DECIMAL(12, 2);    -- PostgreSQL

-- Change column name and type
ALTER TABLE employees
CHANGE old_column_name new_column_name VARCHAR(100);  -- MySQL
-- RENAME COLUMN old_name TO new_name;                -- PostgreSQL

-- Drop column
ALTER TABLE employees
DROP COLUMN middle_name;

-- Add primary key
ALTER TABLE table_name
ADD PRIMARY KEY (column_name);

-- Drop primary key
ALTER TABLE table_name
DROP PRIMARY KEY;  -- MySQL
-- DROP CONSTRAINT constraint_name;  -- PostgreSQL

-- Add foreign key
ALTER TABLE employees
ADD CONSTRAINT fk_dept
    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE CASCADE;

-- Drop foreign key
ALTER TABLE employees
DROP FOREIGN KEY fk_dept;  -- MySQL
-- DROP CONSTRAINT fk_dept;  -- PostgreSQL

-- Add unique constraint
ALTER TABLE employees
ADD CONSTRAINT uq_email UNIQUE (email);

-- Add check constraint
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary >= 0);

-- Add index
ALTER TABLE employees
ADD INDEX idx_last_name (last_name);

-- Rename table
ALTER TABLE old_table_name
RENAME TO new_table_name;

-- Change table storage engine (MySQL)
ALTER TABLE employees ENGINE = InnoDB;

-- Change table character set (MySQL)
ALTER TABLE employees
CONVERT TO CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

### DROP

```sql
-- Drop database
DROP DATABASE database_name;
DROP DATABASE IF EXISTS database_name;

-- Drop table
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables
DROP TABLE IF EXISTS table1, table2, table3;

-- Drop table with dependencies (PostgreSQL)
DROP TABLE table_name CASCADE;

-- Drop view
DROP VIEW view_name;
DROP VIEW IF EXISTS view_name;

-- Drop index
DROP INDEX index_name ON table_name;  -- MySQL
-- DROP INDEX index_name;               -- PostgreSQL

-- Drop constraint
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;
```

### TRUNCATE

```sql
-- Remove all rows (faster than DELETE, resets auto-increment)
TRUNCATE TABLE table_name;

-- PostgreSQL - reset identity and cascade
TRUNCATE TABLE table_name
    RESTART IDENTITY
    CASCADE;
```

### CREATE INDEX

```sql
-- Basic index
CREATE INDEX index_name
ON table_name (column_name);

-- Composite index
CREATE INDEX idx_name_email
ON employees (last_name, first_name, email);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email
ON employees (email);

-- Partial/filtered index (PostgreSQL)
CREATE INDEX idx_active_employees
ON employees (last_name)
WHERE status = 'active';

-- Full-text index (MySQL)
CREATE FULLTEXT INDEX idx_description
ON products (description);

-- Descending index
CREATE INDEX idx_salary_desc
ON employees (salary DESC);

-- Expression-based index (PostgreSQL)
CREATE INDEX idx_lower_email
ON employees (LOWER(email));

-- Drop index
DROP INDEX index_name ON table_name;  -- MySQL
-- DROP INDEX index_name;               -- PostgreSQL
```

### CREATE VIEW

```sql
-- Basic view
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;

-- Complex view with joins
CREATE VIEW employee_details AS
SELECT
    e.employee_id,
    e.first_name,
    e.last_name,
    e.email,
    d.department_name,
    e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE e.status = 'active';

-- View with aggregation
CREATE VIEW department_stats AS
SELECT
    d.department_name,
    COUNT(e.employee_id) as employee_count,
    AVG(e.salary) as avg_salary,
    MAX(e.salary) as max_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;

-- Updatable view (with check option)
CREATE VIEW active_employees AS
SELECT employee_id, first_name, last_name, email
FROM employees
WHERE status = 'active'
WITH CHECK OPTION;

-- Materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    product_id,
    SUM(quantity) as total_quantity,
    SUM(amount) as total_amount
FROM sales
GROUP BY product_id;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW sales_summary;

-- Replace/modify view
CREATE OR REPLACE VIEW view_name AS
SELECT new_columns FROM table_name;

-- Drop view
DROP VIEW view_name;
DROP VIEW IF EXISTS view_name;
```

### CREATE SEQUENCE (PostgreSQL/Oracle)

```sql
-- Create sequence
CREATE SEQUENCE sequence_name
    START WITH 1
    INCREMENT BY 1
    MINVALUE 1
    MAXVALUE 999999
    CACHE 20
    CYCLE;  -- or NO CYCLE

-- Use sequence
SELECT nextval('sequence_name');
SELECT currval('sequence_name');
SELECT setval('sequence_name', 100);

-- Drop sequence
DROP SEQUENCE sequence_name;
```

---

## DQL - Data Query Language

Commands for querying and retrieving data.

### SELECT - Basic Structure

```sql
-- Basic syntax (execution order in comments)
SELECT [DISTINCT] column_list          -- 5. Select columns
FROM table_name                        -- 1. From tables
[JOIN other_table ON condition]        -- 2. Join tables
[WHERE conditions]                     -- 3. Filter rows
[GROUP BY column_list]                 -- 4. Group rows
[HAVING group_conditions]              -- 6. Filter groups
[ORDER BY column_list [ASC|DESC]]      -- 7. Sort results
[LIMIT count [OFFSET offset]];         -- 8. Limit results
```

### SELECT - Examples

```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, email FROM employees;

-- Select with alias
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary * 12 AS annual_salary
FROM employees;

-- Select distinct values
SELECT DISTINCT department_id FROM employees;

-- Select with literal values
SELECT
    first_name,
    last_name,
    'Active' AS status,
    100 AS bonus_points
FROM employees;

-- Select with calculations
SELECT
    product_name,
    price,
    quantity,
    price * quantity AS total_value,
    price * quantity * 0.1 AS tax
FROM products;
```

### WHERE Clause

```sql
-- Comparison operators
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE salary < 50000;
SELECT * FROM employees WHERE salary <= 50000;
SELECT * FROM employees WHERE salary = 50000;
SELECT * FROM employees WHERE salary != 50000;  -- or <>

-- Logical operators
SELECT * FROM employees
WHERE salary > 50000 AND department_id = 5;

SELECT * FROM employees
WHERE department_id = 5 OR department_id = 10;

SELECT * FROM employees
WHERE NOT (salary < 30000);

-- BETWEEN
SELECT * FROM employees
WHERE salary BETWEEN 40000 AND 60000;

SELECT * FROM employees
WHERE hire_date BETWEEN '2020-01-01' AND '2023-12-31';

-- IN
SELECT * FROM employees
WHERE department_id IN (5, 10, 15);

SELECT * FROM employees
WHERE first_name IN ('John', 'Jane', 'Bob');

-- NOT IN
SELECT * FROM employees
WHERE department_id NOT IN (5, 10);

-- LIKE (pattern matching)
SELECT * FROM employees WHERE last_name LIKE 'S%';      -- Starts with S
SELECT * FROM employees WHERE last_name LIKE '%son';    -- Ends with son
SELECT * FROM employees WHERE last_name LIKE '%mit%';   -- Contains mit
SELECT * FROM employees WHERE first_name LIKE 'J___';   -- J followed by 3 chars
SELECT * FROM employees WHERE email LIKE '%@gmail.com'; -- Gmail emails

-- ILIKE (case-insensitive, PostgreSQL)
SELECT * FROM employees WHERE last_name ILIKE 's%';

-- REGEXP/RLIKE (MySQL)
SELECT * FROM employees WHERE email REGEXP '^[a-z]+@company\\.com$';

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE phone IS NULL;
SELECT * FROM employees WHERE phone IS NOT NULL;

-- EXISTS
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department_id = d.department_id
);

-- ANY/ALL
SELECT * FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department_id = 5);

SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department_id = 5);
```

### JOIN Operations

```sql
-- INNER JOIN (only matching rows)
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
    e.last_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- FULL OUTER JOIN (all from both, matching where possible)
SELECT
    e.first_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;
-- MySQL doesn't support FULL OUTER JOIN, use UNION:
-- SELECT ... FROM a LEFT JOIN b UNION SELECT ... FROM a RIGHT JOIN b

-- CROSS JOIN (Cartesian product)
SELECT
    e.first_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;

-- SELF JOIN
SELECT
    e1.first_name AS employee,
    e2.first_name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;

-- Multiple joins
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

-- NATURAL JOIN (joins on all columns with same name - not recommended)
SELECT * FROM employees NATURAL JOIN departments;

-- JOIN USING (when column names are the same)
SELECT *
FROM employees e
JOIN departments d USING (department_id);
```

### GROUP BY and Aggregate Functions

```sql
-- Basic aggregate functions
SELECT
    COUNT(*) as total_employees,
    COUNT(DISTINCT department_id) as department_count,
    SUM(salary) as total_salary,
    AVG(salary) as average_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees;

-- GROUP BY single column
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
    COUNT(*) as count
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
    AVG(e.salary) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
ORDER BY employee_count DESC;

-- String aggregation
SELECT
    department_id,
    GROUP_CONCAT(first_name ORDER BY last_name SEPARATOR ', ') as employees  -- MySQL
    -- STRING_AGG(first_name, ', ' ORDER BY last_name) as employees           -- PostgreSQL
FROM employees
GROUP BY department_id;

-- ROLLUP (subtotals and grand total)
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY department_id, status WITH ROLLUP;  -- MySQL
-- GROUP BY ROLLUP(department_id, status);    -- PostgreSQL

-- CUBE (all possible subtotals)
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY CUBE(department_id, status);  -- PostgreSQL, Oracle

-- GROUPING SETS
SELECT
    department_id,
    status,
    COUNT(*) as count
FROM employees
GROUP BY GROUPING SETS (
    (department_id, status),
    (department_id),
    (status),
    ()
);
```

### ORDER BY

```sql
-- Order by single column (ascending by default)
SELECT * FROM employees ORDER BY last_name;
SELECT * FROM employees ORDER BY last_name ASC;

-- Order by descending
SELECT * FROM employees ORDER BY salary DESC;

-- Order by multiple columns
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;

-- Order by column position (not recommended)
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;  -- Orders by salary (3rd column)

-- Order by expression
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary * 12 DESC;

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
    END;

-- NULL handling
SELECT * FROM employees
ORDER BY phone NULLS FIRST;   -- PostgreSQL
-- ORDER BY phone NULLS LAST;  -- PostgreSQL

-- MySQL NULL handling (NULL values come first in ASC, last in DESC)
SELECT * FROM employees
ORDER BY ISNULL(phone), phone;  -- NULLs last in ASC
```

### LIMIT and OFFSET

```sql
-- LIMIT (restrict number of rows)
SELECT * FROM employees LIMIT 10;

-- LIMIT with OFFSET (pagination)
SELECT * FROM employees
LIMIT 10 OFFSET 20;  -- Skip 20, return next 10 (rows 21-30)

-- Alternative syntax (MySQL)
SELECT * FROM employees LIMIT 20, 10;  -- Offset 20, limit 10

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
-- Subquery in WHERE clause
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) FROM employees
);

-- Subquery with IN
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id = 100
);

-- Subquery with EXISTS
SELECT d.department_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);

-- Subquery in SELECT (scalar subquery)
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

-- Correlated subquery
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
```

### Common Table Expressions (CTE)

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
SELECT * FROM employee_hierarchy
ORDER BY level, last_name;

-- Recursive CTE for hierarchical sums
WITH RECURSIVE category_tree AS (
    SELECT
        category_id,
        parent_id,
        category_name,
        revenue,
        revenue as total_revenue
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT
        c.category_id,
        c.parent_id,
        c.category_name,
        c.revenue,
        c.revenue
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT
    category_name,
    SUM(total_revenue) as total_with_children
FROM category_tree
GROUP BY category_id, category_name;
```

### Window Functions

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

-- Frame specifications
-- ROWS - physical rows
-- RANGE - logical range based on values
SELECT
    employee_id,
    salary,
    AVG(salary) OVER (
        ORDER BY salary
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) as avg_nearby_rows,
    AVG(salary) OVER (
        ORDER BY salary
        RANGE BETWEEN 5000 PRECEDING AND 5000 FOLLOWING
    ) as avg_nearby_range
FROM employees;
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

-- INTERSECT (common rows in both queries)
SELECT employee_id FROM employees
INTERSECT
SELECT employee_id FROM project_members;

-- EXCEPT/MINUS (rows in first query but not in second)
SELECT employee_id FROM employees
EXCEPT  -- PostgreSQL, SQL Server
-- MINUS  -- Oracle, MySQL 8.0.31+
SELECT employee_id FROM contractors;

-- Complex UNION example
(
    SELECT
        'Employee' as type,
        first_name,
        last_name,
        email
    FROM employees
    WHERE status = 'active'
)
UNION ALL
(
    SELECT
        'Contractor' as type,
        first_name,
        last_name,
        email
    FROM contractors
    WHERE end_date > CURRENT_DATE
)
ORDER BY type, last_name;
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
    SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END) as active_count
FROM employees
GROUP BY department_id;

-- CASE in ORDER BY
SELECT first_name, last_name, status
FROM employees
ORDER BY
    CASE status
        WHEN 'active' THEN 1
        WHEN 'inactive' THEN 2
        ELSE 3
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

Commands for inserting, updating, and deleting data.

### INSERT

```sql
-- Insert single row (specify columns)
INSERT INTO employees (first_name, last_name, email, hire_date, salary)
VALUES ('John', 'Doe', 'john.doe@company.com', '2024-01-15', 55000.00);

-- Insert single row (all columns, in order)
INSERT INTO employees
VALUES (101, 'Jane', 'Smith', 'jane.smith@company.com', '555-0123',
        '2024-01-20', '1990-05-15', CURRENT_TIMESTAMP, CURRENT_TIMESTAMP,
        60000.00, 5000.00, 5, 50, 'active', TRUE);

-- Insert multiple rows
INSERT INTO employees (first_name, last_name, email, salary)
VALUES
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

-- Insert with subquery
INSERT INTO high_earners (employee_id, name, salary)
SELECT
    employee_id,
    CONCAT(first_name, ' ', last_name),
    salary
FROM employees
WHERE salary > 75000;

-- Insert and return values (PostgreSQL)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Tom', 'Wilson', 'tom@company.com')
RETURNING employee_id, first_name, created_at;

-- Insert and get last ID (MySQL)
INSERT INTO employees (first_name, last_name, email)
VALUES ('Lisa', 'Anderson', 'lisa@company.com');
SELECT LAST_INSERT_ID();

-- Insert on duplicate key update (MySQL)
INSERT INTO employees (employee_id, first_name, last_name, email, salary)
VALUES (101, 'John', 'Doe', 'john@company.com', 55000)
ON DUPLICATE KEY UPDATE
    first_name = VALUES(first_name),
    last_name = VALUES(last_name),
    salary = VALUES(salary),
    updated_at = CURRENT_TIMESTAMP;

-- Insert on conflict (PostgreSQL - UPSERT)
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

-- Update all rows (no WHERE - be careful!)
UPDATE employees
SET updated_at = CURRENT_TIMESTAMP;

-- Update with LIMIT (MySQL - update only N rows)
UPDATE employees
SET status = 'inactive'
WHERE last_login < '2020-01-01'
ORDER BY last_login
LIMIT 100;

-- Update and return (PostgreSQL)
UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 5
RETURNING employee_id, first_name, salary;

-- Update with multiple table references (MySQL)
UPDATE employees e, departments d
SET e.location = d.location
WHERE e.department_id = d.department_id;

-- Conditional update
UPDATE products
SET
    stock_status = CASE
        WHEN quantity = 0 THEN 'Out of Stock'
        WHEN quantity < 10 THEN 'Low Stock'
        ELSE 'In Stock'
    END,
    last_checked = CURRENT_TIMESTAMP;
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

-- Delete all rows (use TRUNCATE instead for better performance)
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

-- Self-referencing delete (keep only most recent)
DELETE e1
FROM employees e1
INNER JOIN employees e2
    ON e1.email = e2.email
    AND e1.created_at < e2.created_at;
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

-- Oracle MERGE
MERGE INTO employees e
USING temp_employees t
ON (e.employee_id = t.employee_id)
WHEN MATCHED THEN
    UPDATE SET
        e.salary = t.salary,
        e.updated_at = SYSDATE
    WHERE e.salary != t.salary
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, email, salary)
    VALUES (t.employee_id, t.first_name, t.last_name, t.email, t.salary);

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

Commands for controlling access to data.

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

-- Grant CREATE permission
GRANT CREATE TABLE TO user_name;
GRANT CREATE VIEW TO user_name;

-- Grant role to user (PostgreSQL, Oracle)
GRANT role_name TO user_name;

-- Grant database-level permissions (MySQL)
GRANT CREATE, DROP, ALTER ON database_name.* TO user_name;

-- Grant with conditions (PostgreSQL row-level security)
CREATE POLICY employee_policy ON employees
    FOR SELECT
    TO regular_user
    USING (department_id = current_setting('app.current_department_id')::int);

ALTER TABLE employees ENABLE ROW LEVEL SECURITY;
```

### REVOKE

```sql
-- Revoke SELECT permission
REVOKE SELECT ON employees FROM user_name;

-- Revoke multiple permissions
REVOKE SELECT, INSERT, UPDATE ON employees FROM user_name;

-- Revoke all permissions
REVOKE ALL PRIVILEGES ON employees FROM user_name;

-- Revoke with CASCADE (also revoke from users who got it from this user)
REVOKE SELECT ON employees FROM user_name CASCADE;

-- Revoke GRANT OPTION only
REVOKE GRANT OPTION FOR SELECT ON employees FROM user_name;

-- Revoke role from user
REVOKE role_name FROM user_name;

-- Revoke column-specific permissions
REVOKE UPDATE (salary) ON employees FROM user_name;

-- Revoke EXECUTE permission
REVOKE EXECUTE ON PROCEDURE procedure_name FROM user_name;
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
CREATE ROLE role_name;
CREATE ROLE readonly_role;
CREATE ROLE admin_role WITH LOGIN PASSWORD 'password' CREATEDB;

-- Alter user
ALTER USER username WITH PASSWORD 'new_password';

-- Drop user
DROP USER username;
DROP USER IF EXISTS username;

-- Create role with permissions (PostgreSQL)
CREATE ROLE app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;
GRANT app_readonly TO username;

-- Show grants
SHOW GRANTS FOR username;  -- MySQL
-- \du username               -- PostgreSQL
```

---

## TCL - Transaction Control Language

Commands for managing transactions.

### Transaction Basics

```sql
-- Begin transaction
BEGIN;  -- PostgreSQL, MySQL (in some modes)
-- BEGIN TRANSACTION;  -- SQL Server
-- START TRANSACTION;  -- MySQL standard

-- Commit transaction (save changes)
COMMIT;

-- Rollback transaction (undo changes)
ROLLBACK;

-- Basic transaction example
BEGIN;

INSERT INTO accounts (account_id, balance)
VALUES (1, 1000), (2, 500);

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;  -- or ROLLBACK if something went wrong
```

### SAVEPOINT

```sql
-- Create savepoint
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

### Transaction Properties Example

```sql
-- Bank transfer example
BEGIN;

-- Deduct from source account
UPDATE accounts
SET balance = balance - 500
WHERE account_id = 1;

-- Verify balance isn't negative
DO $$
BEGIN
    IF (SELECT balance FROM accounts WHERE account_id = 1) < 0 THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;
END $$;

-- Add to destination account
UPDATE accounts
SET balance = balance + 500
WHERE account_id = 2;

-- Record transaction
INSERT INTO transactions (from_account, to_account, amount, timestamp)
VALUES (1, 2, 500, CURRENT_TIMESTAMP);

COMMIT;
```

### Isolation Levels

```sql
-- Set isolation level for next transaction (MySQL)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set isolation level globally (MySQL)
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Set isolation level for session
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- PostgreSQL syntax
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- SQL Server syntax
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
-- ... queries ...
COMMIT;

-- Check current isolation level
SELECT @@transaction_isolation;  -- MySQL
-- SHOW TRANSACTION ISOLATION LEVEL;  -- PostgreSQL
```

### Transaction Modes

```sql
-- Read-only transaction
BEGIN TRANSACTION READ ONLY;
SELECT * FROM employees;
COMMIT;

-- Read-write transaction (default)
BEGIN TRANSACTION READ WRITE;
UPDATE employees SET salary = salary * 1.1;
COMMIT;

-- Deferrable transaction (PostgreSQL)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE, READ ONLY, DEFERRABLE;
-- Long-running read-only query
COMMIT;
```

### Autocommit

```sql
-- Disable autocommit (MySQL)
SET autocommit = 0;
-- Now each statement requires explicit COMMIT

-- Enable autocommit
SET autocommit = 1;

-- PostgreSQL uses autocommit by default unless in BEGIN block

-- SQL Server
SET IMPLICIT_TRANSACTIONS ON;  -- Disable autocommit
SET IMPLICIT_TRANSACTIONS OFF;  -- Enable autocommit
```

### Lock Management

```sql
-- Explicit table locks (MySQL)
LOCK TABLES employees WRITE;
-- ... perform operations ...
UNLOCK TABLES;

LOCK TABLES employees READ;
-- ... read operations only ...
UNLOCK TABLES;

-- Row-level locks (PostgreSQL)
BEGIN;

-- Lock row for update
SELECT * FROM employees
WHERE employee_id = 101
FOR UPDATE;

-- Lock row for share (other transactions can read but not update)
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

-- Advisory locks (PostgreSQL)
SELECT pg_advisory_lock(123456);
-- ... critical section ...
SELECT pg_advisory_unlock(123456);
```

---

## Procedural SQL

### Variables (PostgreSQL - PL/pgSQL)

```sql
-- Declare variables in a DO block
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

-- Session variables
SET SESSION sql_mode = 'STRICT_TRANS_TABLES';
SELECT @@sql_mode;

-- Global variables
SET GLOBAL max_connections = 200;
SELECT @@global.max_connections;
```

### Variables (SQL Server - T-SQL)

```sql
-- Declare variables
DECLARE @employee_id INT = 101;
DECLARE @first_name VARCHAR(50);
DECLARE @last_name VARCHAR(50);
DECLARE @salary DECIMAL(10, 2);
DECLARE @count INT;

-- Multiple declarations
DECLARE
    @var1 INT = 1,
    @var2 VARCHAR(50) = 'Test',
    @var3 DATE = GETDATE();

-- Assign values
SET @first_name = 'John';
SET @last_name = 'Doe';

-- Select into variables
SELECT
    @first_name = first_name,
    @last_name = last_name,
    @salary = salary
FROM employees
WHERE employee_id = @employee_id;

-- Use variables
SELECT @first_name AS FirstName,
       @last_name AS LastName,
       @salary AS Salary;

PRINT 'Employee: ' + @first_name + ' ' + @last_name;
```

### IF-ELSE Statements

```sql
-- PostgreSQL (PL/pgSQL)
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

    -- Nested IF
    IF v_salary > 50000 THEN
        IF v_salary > 80000 THEN
            v_bonus := v_salary * 0.25;
        ELSE
            v_bonus := v_salary * 0.15;
        END IF;
    ELSE
        v_bonus := v_salary * 0.08;
    END IF;

    RAISE NOTICE 'Bonus: %', v_bonus;
END $$;

-- MySQL (stored procedure required for IF)
DELIMITER //
CREATE PROCEDURE calculate_bonus(IN emp_id INT)
BEGIN
    DECLARE v_salary DECIMAL(10, 2);
    DECLARE v_bonus DECIMAL(10, 2);

    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = emp_id;

    IF v_salary > 80000 THEN
        SET v_bonus = v_salary * 0.20;
    ELSEIF v_salary > 60000 THEN
        SET v_bonus = v_salary * 0.15;
    ELSEIF v_salary > 40000 THEN
        SET v_bonus = v_salary * 0.10;
    ELSE
        SET v_bonus = v_salary * 0.05;
    END IF;

    SELECT v_bonus AS bonus;
END //
DELIMITER ;

-- SQL Server (T-SQL)
DECLARE @salary DECIMAL(10, 2);
DECLARE @bonus DECIMAL(10, 2);

SELECT @salary = salary
FROM employees
WHERE employee_id = 101;

-- Simple IF
IF @salary > 60000
    SET @bonus = @salary * 0.15;

-- IF-ELSE
IF @salary > 60000
    SET @bonus = @salary * 0.15;
ELSE
    SET @bonus = @salary * 0.10;

-- IF-ELSE IF
IF @salary > 80000
    SET @bonus = @salary * 0.20;
ELSE IF @salary > 60000
    SET @bonus = @salary * 0.15;
ELSE IF @salary > 40000
    SET @bonus = @salary * 0.10;
ELSE
    SET @bonus = @salary * 0.05;

-- IF with BEGIN-END block
IF @salary > 60000
BEGIN
    SET @bonus = @salary * 0.15;
    PRINT 'High earner bonus applied';
END
ELSE
BEGIN
    SET @bonus = @salary * 0.10;
    PRINT 'Standard bonus applied';
END

-- IF EXISTS
IF EXISTS (SELECT 1 FROM employees WHERE salary > 100000)
BEGIN
    PRINT 'High earners found';
END
```

### CASE in Procedural Code

```sql
-- PostgreSQL CASE expression
DO $$
DECLARE
    v_dept_id INT := 5;
    v_dept_name VARCHAR(50);
    v_salary NUMERIC := 55000;
    v_grade CHAR(1);
BEGIN
    -- Simple CASE
    v_dept_name := CASE v_dept_id
        WHEN 1 THEN 'Sales'
        WHEN 2 THEN 'Marketing'
        WHEN 3 THEN 'Engineering'
        ELSE 'Other'
    END;

    -- Searched CASE
    v_grade := CASE
        WHEN v_salary > 80000 THEN 'A'
        WHEN v_salary > 60000 THEN 'B'
        WHEN v_salary > 40000 THEN 'C'
        ELSE 'D'
    END;

    RAISE NOTICE 'Department: %, Grade: %', v_dept_name, v_grade;
END $$;

-- SQL Server CASE
DECLARE @dept_id INT = 5;
DECLARE @dept_name VARCHAR(50);

SET @dept_name = CASE @dept_id
    WHEN 1 THEN 'Sales'
    WHEN 2 THEN 'Marketing'
    WHEN 3 THEN 'Engineering'
    ELSE 'Other'
END;

SELECT @dept_name AS DepartmentName;
```

### WHILE Loops

```sql
-- PostgreSQL WHILE
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

-- MySQL WHILE
DELIMITER //
CREATE PROCEDURE sum_numbers()
BEGIN
    DECLARE v_counter INT DEFAULT 1;
    DECLARE v_sum INT DEFAULT 0;

    WHILE v_counter <= 10 DO
        SET v_sum = v_sum + v_counter;
        SET v_counter = v_counter + 1;

        -- No direct EXIT in MySQL, use IF with LEAVE
        IF v_sum > 30 THEN
            LEAVE;  -- Requires label
        END IF;
    END WHILE;

    SELECT v_sum AS total_sum;
END //
DELIMITER ;

-- MySQL WHILE with label
DELIMITER //
CREATE PROCEDURE labeled_loop()
BEGIN
    DECLARE v_i INT DEFAULT 1;

    my_loop: WHILE v_i <= 10 DO
        SET v_i = v_i + 1;

        IF v_i = 5 THEN
            ITERATE my_loop;  -- Continue
        END IF;

        IF v_i = 8 THEN
            LEAVE my_loop;  -- Break
        END IF;

        SELECT v_i;
    END WHILE my_loop;
END //
DELIMITER ;

-- SQL Server WHILE
DECLARE @counter INT = 1;
DECLARE @sum INT = 0;

WHILE @counter <= 10
BEGIN
    SET @sum = @sum + @counter;
    SET @counter = @counter + 1;

    -- Exit condition
    IF @sum > 30
        BREAK;

    -- Continue to next iteration
    IF @counter = 5
        CONTINUE;

    PRINT 'Counter: ' + CAST(@counter AS VARCHAR) +
          ', Sum: ' + CAST(@sum AS VARCHAR);
END

PRINT 'Final sum: ' + CAST(@sum AS VARCHAR);
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

-- MySQL doesn't have FOR loops, use WHILE or CURSOR
-- SQL Server doesn't have FOR loops, use WHILE or CURSOR
```

### LOOP (PostgreSQL)

```sql
-- Basic LOOP (infinite loop with EXIT)
DO $$
DECLARE
    v_counter INT := 1;
BEGIN
    LOOP
        RAISE NOTICE 'Counter: %', v_counter;
        v_counter := v_counter + 1;

        -- Exit condition
        EXIT WHEN v_counter > 10;
    END LOOP;
END $$;

-- Labeled LOOP
DO $$
DECLARE
    v_i INT := 1;
    v_j INT;
BEGIN
    <<outer_loop>>
    LOOP
        v_j := 1;
        RAISE NOTICE 'Outer: %', v_i;

        <<inner_loop>>
        LOOP
            RAISE NOTICE '  Inner: %', v_j;
            v_j := v_j + 1;

            EXIT inner_loop WHEN v_j > 3;
        END LOOP inner_loop;

        v_i := v_i + 1;
        EXIT outer_loop WHEN v_i > 3;
    END LOOP outer_loop;
END $$;
```

### REPEAT (MySQL)

```sql
-- MySQL REPEAT loop
DELIMITER //
CREATE PROCEDURE repeat_example()
BEGIN
    DECLARE v_counter INT DEFAULT 1;

    REPEAT
        SELECT v_counter;
        SET v_counter = v_counter + 1;
    UNTIL v_counter > 10
    END REPEAT;
END //
DELIMITER ;
```

### Cursors

```sql
-- PostgreSQL Cursor
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

-- PostgreSQL Cursor with FOR loop (simpler)
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

-- MySQL Cursor
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

-- SQL Server Cursor
DECLARE @emp_id INT;
DECLARE @first_name VARCHAR(50);
DECLARE @salary DECIMAL(10, 2);

-- Declare cursor
DECLARE emp_cursor CURSOR FOR
    SELECT employee_id, first_name, salary
    FROM employees
    WHERE department_id = 5;

-- Open cursor
OPEN emp_cursor;

-- Fetch first row
FETCH NEXT FROM emp_cursor
INTO @emp_id, @first_name, @salary;

-- Fetch loop
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process row
    PRINT 'Employee: ' + @first_name + ', Salary: ' + CAST(@salary AS VARCHAR);

    UPDATE employees
    SET salary = salary * 1.05
    WHERE employee_id = @emp_id;

    -- Fetch next row
    FETCH NEXT FROM emp_cursor
    INTO @emp_id, @first_name, @salary;
END

-- Close and deallocate cursor
CLOSE emp_cursor;
DEALLOCATE emp_cursor;
```

### Functions

```sql
-- PostgreSQL Function
CREATE OR REPLACE FUNCTION calculate_bonus(
    p_employee_id INT
) RETURNS NUMERIC AS $$
DECLARE
    v_salary NUMERIC;
    v_bonus NUMERIC;
BEGIN
    -- Get salary
    SELECT salary INTO v_salary
    FROM employees
    WHERE employee_id = p_employee_id;

    -- Calculate bonus
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

-- PostgreSQL Function returning table
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

-- PostgreSQL Function with OUT parameters
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

-- MySQL Function
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

-- SQL Server Function
CREATE FUNCTION dbo.CalculateBonus(
    @employee_id INT
)
RETURNS DECIMAL(10, 2)
AS
BEGIN
    DECLARE @salary DECIMAL(10, 2);
    DECLARE @bonus DECIMAL(10, 2);

    SELECT @salary = salary
    FROM employees
    WHERE employee_id = @employee_id;

    IF @salary > 80000
        SET @bonus = @salary * 0.20;
    ELSE IF @salary > 60000
        SET @bonus = @salary * 0.15;
    ELSE
        SET @bonus = @salary * 0.10;

    RETURN @bonus;
END;

-- Call function
SELECT dbo.CalculateBonus(101);

-- SQL Server Table-valued function
CREATE FUNCTION dbo.GetHighEarners(
    @min_salary DECIMAL(10, 2)
)
RETURNS TABLE
AS
RETURN
(
    SELECT
        employee_id,
        first_name + ' ' + last_name AS full_name,
        salary
    FROM employees
    WHERE salary >= @min_salary
);

-- Call table function
SELECT * FROM dbo.GetHighEarners(70000);

-- Drop function
DROP FUNCTION calculate_bonus;
DROP FUNCTION IF EXISTS calculate_bonus;  -- PostgreSQL
```

### Stored Procedures

```sql
-- PostgreSQL Procedure
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

-- PostgreSQL Procedure with INOUT parameters
CREATE OR REPLACE PROCEDURE process_order(
    p_order_id INT,
    INOUT p_status VARCHAR,
    OUT p_total NUMERIC
) AS $$
BEGIN
    -- Process order logic
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

-- MySQL Stored Procedure
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

-- SQL Server Stored Procedure
CREATE PROCEDURE GiveRaise
    @employee_id INT,
    @percentage DECIMAL(5, 2),
    @old_salary DECIMAL(10, 2) OUTPUT,
    @new_salary DECIMAL(10, 2) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    -- Get current salary
    SELECT @old_salary = salary
    FROM employees
    WHERE employee_id = @employee_id;

    -- Calculate new salary
    SET @new_salary = @old_salary * (1 + @percentage / 100);

    -- Update salary
    UPDATE employees
    SET salary = @new_salary,
        updated_at = GETDATE()
    WHERE employee_id = @employee_id;

    -- Log the change
    INSERT INTO salary_history (employee_id, old_salary, new_salary, change_date)
    VALUES (@employee_id, @old_salary, @new_salary, GETDATE());
END;

-- Call procedure
DECLARE @old DECIMAL(10, 2), @new DECIMAL(10, 2);
EXEC GiveRaise 101, 10, @old OUTPUT, @new OUTPUT;
SELECT @old AS old_salary, @new AS new_salary;

-- Drop procedure
DROP PROCEDURE give_raise;
DROP PROCEDURE IF EXISTS give_raise;  -- PostgreSQL, MySQL
```

### Triggers

```sql
-- PostgreSQL Trigger (requires function)
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

-- BEFORE trigger example
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

-- MySQL Trigger
DELIMITER //

-- AFTER INSERT trigger
CREATE TRIGGER employee_after_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (operation, employee_id, new_salary, changed_at)
    VALUES ('INSERT', NEW.employee_id, NEW.salary, NOW());
END //

-- AFTER UPDATE trigger
CREATE TRIGGER employee_after_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (
        operation, employee_id, old_salary, new_salary, changed_at
    )
    VALUES (
        'UPDATE', NEW.employee_id, OLD.salary, NEW.salary, NOW()
    );
END //

-- BEFORE INSERT trigger
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

-- SQL Server Trigger
-- AFTER INSERT trigger
CREATE TRIGGER employee_after_insert
ON employees
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO employee_audit (operation, employee_id, new_salary, changed_at)
    SELECT 'INSERT', employee_id, salary, GETDATE()
    FROM inserted;
END;

-- AFTER UPDATE trigger
CREATE TRIGGER employee_after_update
ON employees
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO employee_audit (
        operation, employee_id, old_salary, new_salary, changed_at
    )
    SELECT
        'UPDATE',
        i.employee_id,
        d.salary,
        i.salary,
        GETDATE()
    FROM inserted i
    INNER JOIN deleted d ON i.employee_id = d.employee_id;
END;

-- INSTEAD OF trigger (replaces the operation)
CREATE TRIGGER employee_instead_of_delete
ON employees
INSTEAD OF DELETE
AS
BEGIN
    SET NOCOUNT ON;

    -- Soft delete instead of hard delete
    UPDATE employees
    SET status = 'deleted',
        deleted_at = GETDATE()
    FROM employees e
    INNER JOIN deleted d ON e.employee_id = d.employee_id;
END;

-- Drop trigger
DROP TRIGGER IF EXISTS employee_audit_trigger;  -- PostgreSQL
-- DROP TRIGGER employee_after_insert;           -- MySQL, SQL Server
```

### Exception Handling

```sql
-- PostgreSQL Exception handling
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
END $$;

-- PostgreSQL with custom exception
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
        RAISE NOTICE 'Error detail: %', SQLSTATE;
        -- Re-raise the exception
        RAISE;
END $$;

-- PostgreSQL nested exception blocks
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

-- MySQL Error handling
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
DELIMITER ;

-- MySQL specific error handlers
DELIMITER //
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

-- SQL Server Error handling (TRY-CATCH)
BEGIN TRY
    BEGIN TRANSACTION;

    -- Transfer money
    UPDATE accounts
    SET balance = balance - 500
    WHERE account_id = 1;

    UPDATE accounts
    SET balance = balance + 500
    WHERE account_id = 2;

    COMMIT TRANSACTION;
    PRINT 'Transaction successful';
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    -- Get error information
    SELECT
        ERROR_NUMBER() AS ErrorNumber,
        ERROR_STATE() AS ErrorState,
        ERROR_SEVERITY() AS ErrorSeverity,
        ERROR_PROCEDURE() AS ErrorProcedure,
        ERROR_LINE() AS ErrorLine,
        ERROR_MESSAGE() AS ErrorMessage;

    -- Re-throw error
    THROW;
END CATCH;

-- SQL Server with RAISERROR
BEGIN TRY
    DECLARE @balance DECIMAL(10, 2);

    SELECT @balance = balance
    FROM accounts
    WHERE account_id = 1;

    IF @balance < 500
    BEGIN
        RAISERROR('Insufficient funds', 16, 1);
    END

    -- Continue with transaction...
END TRY
BEGIN CATCH
    PRINT ERROR_MESSAGE();
END CATCH;
```

---

## Advanced Features

### JSON Operations

```sql
-- PostgreSQL JSON
-- Create table with JSON column
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB  -- JSONB is binary, more efficient than JSON
);

-- Insert JSON data
INSERT INTO products (name, attributes)
VALUES
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

-- MySQL JSON (MySQL 5.7+)
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

-- SQL Server JSON (SQL Server 2016+)
-- Query JSON string
DECLARE @json NVARCHAR(MAX) = '{"brand": "Dell", "ram": 16, "storage": 512}';

SELECT
    JSON_VALUE(@json, '$.brand') AS brand,
    JSON_VALUE(@json, '$.ram') AS ram;

-- Query JSON array
DECLARE @jsonArray NVARCHAR(MAX) = '[
    {"id": 1, "name": "Product 1"},
    {"id": 2, "name": "Product 2"}
]';

SELECT *
FROM OPENJSON(@jsonArray)
WITH (
    id INT '$.id',
    name VARCHAR(100) '$.name'
);

-- Convert result to JSON
SELECT
    employee_id,
    first_name,
    last_name
FROM employees
FOR JSON PATH;

-- Generate JSON
SELECT
    department_id,
    (
        SELECT employee_id, first_name, last_name
        FROM employees e
        WHERE e.department_id = d.department_id
        FOR JSON PATH
    ) AS employees
FROM departments d
FOR JSON PATH;
```

### Full-Text Search

```sql
-- MySQL Full-Text Search
-- Create table with FULLTEXT index
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    FULLTEXT KEY ft_title_content (title, content)
);

-- Natural language search
SELECT *
FROM articles
WHERE MATCH(title, content) AGAINST('database performance');

-- Boolean mode search
SELECT *
FROM articles
WHERE MATCH(title, content)
AGAINST('+database -mysql' IN BOOLEAN MODE);

-- Search operators:
-- + : must include
-- - : must not include
-- * : wildcard
-- " " : exact phrase
-- > < : increase/decrease relevance

-- Query expansion
SELECT *
FROM articles
WHERE MATCH(title, content)
AGAINST('database' WITH QUERY EXPANSION);

-- PostgreSQL Full-Text Search
-- Simple text search
SELECT *
FROM articles
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

### Partitioning

```sql
-- MySQL Partitioning (RANGE)
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

-- MySQL LIST partitioning
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

-- MySQL HASH partitioning
CREATE TABLE customers (
    id INT,
    name VARCHAR(100)
)
PARTITION BY HASH (id)
PARTITIONS 4;

-- PostgreSQL Declarative Partitioning (PostgreSQL 10+)
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

### Recursive Queries (Hierarchical Data)

```sql
-- Employee hierarchy
WITH RECURSIVE employee_tree AS (
    -- Anchor: top-level employees (no manager)
    SELECT
        employee_id,
        first_name,
        last_name,
        manager_id,
        1 AS level,
        first_name || ' ' || last_name AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: employees with managers
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        et.level + 1,
        et.path || ' -> ' || e.first_name || ' ' || e.last_name
    FROM employees e
    INNER JOIN employee_tree et ON e.manager_id = et.employee_id
)
SELECT
    REPEAT('  ', level - 1) || first_name || ' ' || last_name AS employee,
    level,
    path
FROM employee_tree
ORDER BY path;

-- Category tree with totals
WITH RECURSIVE category_totals AS (
    -- Leaf categories
    SELECT
        category_id,
        parent_id,
        name,
        revenue,
        revenue AS total_revenue
    FROM categories
    WHERE category_id NOT IN (SELECT DISTINCT parent_id FROM categories WHERE parent_id IS NOT NULL)

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

### Pivot and Unpivot

```sql
-- Pivot example (rows to columns)
-- MySQL doesn't have built-in PIVOT, use CASE
SELECT
    employee_id,
    MAX(CASE WHEN skill = 'SQL' THEN proficiency END) AS SQL,
    MAX(CASE WHEN skill = 'Python' THEN proficiency END) AS Python,
    MAX(CASE WHEN skill = 'Java' THEN proficiency END) AS Java
FROM employee_skills
GROUP BY employee_id;

-- PostgreSQL crosstab (requires tablefunc extension)
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT *
FROM crosstab(
    'SELECT employee_id, skill, proficiency
     FROM employee_skills
     ORDER BY 1, 2',
    'SELECT DISTINCT skill FROM employee_skills ORDER BY 1'
) AS ct(employee_id INT, SQL INT, Python INT, Java INT);

-- SQL Server PIVOT
SELECT employee_id, [SQL], [Python], [Java]
FROM (
    SELECT employee_id, skill, proficiency
    FROM employee_skills
) AS source
PIVOT (
    MAX(proficiency)
    FOR skill IN ([SQL], [Python], [Java])
) AS pivot_table;

-- UNPIVOT example (columns to rows)
-- SQL Server
SELECT employee_id, skill, proficiency
FROM (
    SELECT employee_id, SQL, Python, Java
    FROM employee_skills_wide
) AS source
UNPIVOT (
    proficiency FOR skill IN ([SQL], [Python], [Java])
) AS unpivot_table;

-- MySQL/PostgreSQL - use UNION
SELECT employee_id, 'SQL' AS skill, SQL AS proficiency
FROM employee_skills_wide
WHERE SQL IS NOT NULL
UNION ALL
SELECT employee_id, 'Python', Python
FROM employee_skills_wide
WHERE Python IS NOT NULL
UNION ALL
SELECT employee_id, 'Java', Java
FROM employee_skills_wide
WHERE Java IS NOT NULL;
```

---

## Data Types Quick Reference

```sql
-- Numeric types
INT, INTEGER                 -- Whole numbers
SMALLINT                     -- Small integers
BIGINT                       -- Large integers
DECIMAL(p, s), NUMERIC(p, s) -- Fixed precision (p=precision, s=scale)
FLOAT, REAL                  -- Approximate numbers
DOUBLE PRECISION             -- Double precision float

-- String types
CHAR(n)                      -- Fixed length
VARCHAR(n)                   -- Variable length
TEXT                         -- Unlimited text (PostgreSQL, MySQL)
NVARCHAR(n)                  -- Unicode variable (SQL Server)

-- Date/Time types
DATE                         -- Date only
TIME                         -- Time only
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
JSON, JSONB                  -- JSON data (PostgreSQL, MySQL)
XML                          -- XML data
UUID                         -- Universally unique identifier (PostgreSQL)
ENUM('val1', 'val2')         -- Enumerated type (MySQL, PostgreSQL)
ARRAY                        -- Arrays (PostgreSQL)
SERIAL, BIGSERIAL            -- Auto-increment (PostgreSQL)
AUTO_INCREMENT               -- Auto-increment (MySQL)
IDENTITY                     -- Auto-increment (SQL Server)
```

---

This cheatsheet covers the complete SQL landscape across major database systems (PostgreSQL, MySQL, SQL Server). Save it for quick reference!
