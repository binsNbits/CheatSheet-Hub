# PostgreSQL (psql) Cheatsheet - The Programmer's Guide to SQL

A comprehensive reference for developers learning SQL through PostgreSQL. Focused on understanding command syntax, order, and practical usage patterns.

---

## Table of Contents
1. [Connection & Session Management](#connection--session-management)
2. [psql Meta-Commands (Backslash Commands)](#psql-meta-commands-backslash-commands)
3. [SQL Command Structure & Order](#sql-command-structure--order)
4. [Querying Data (SELECT)](#querying-data-select)
5. [Inserting Data (INSERT)](#inserting-data-insert)
6. [Updating Data (UPDATE)](#updating-data-update)
7. [Deleting Data (DELETE)](#deleting-data-delete)
8. [Schema Creation (CREATE TABLE)](#schema-creation-create-table)
9. [Schema Modification (ALTER TABLE)](#schema-modification-alter-table)
10. [Constraints & Relationships](#constraints--relationships)
11. [Indexes](#indexes)
12. [Transactions](#transactions)
13. [Common Patterns & Anti-Patterns](#common-patterns--anti-patterns)

---

## Connection & Session Management

### Basic Connection
```bash
psql
```
**Why:** Connects using your system username as both the database name and user.
**When:** Quick local development when your user/db match your system account.
**Problem:** Often fails with "database does not exist" on fresh installs.

### Full Connection String
```bash
psql -U username -d database_name -h localhost -p 5432
```
**Arguments:**
- `-U username` - User to connect as (default: system user)
- `-d database_name` - Database to connect to (default: username)
- `-h hostname` - Host to connect to (default: localhost via socket)
- `-p port` - Port number (default: 5432)
- `-W` - Force password prompt (useful for remote connections)

**Why:** Full control over connection parameters.
**When:** Connecting to specific databases, remote servers, or with different credentials.
**Example:**
```bash
psql -U admin -d production_db -h db.example.com -p 5432 -W
```

### Connection String Format
```bash
psql postgresql://username:password@localhost:5432/database_name
```
**Why:** Single-string format, useful for environment variables and scripts.
**When:** Deploying apps that need database URLs (common in web frameworks).

### Execute Query and Exit
```bash
psql -U user -d database -c "SELECT * FROM users LIMIT 5;"
```
**Argument:** `-c "SQL_COMMAND"`
**Why:** Run a query without entering interactive mode.
**When:** Scripting, CI/CD pipelines, quick checks.

### Execute SQL File
```bash
psql -U user -d database -f script.sql
```
**Argument:** `-f filename.sql`
**Why:** Run migrations, seed data, or batch operations.
**When:** Database setup, schema changes, importing test data.

---

## psql Meta-Commands (Backslash Commands)

These are psql-specific commands (not SQL). They help navigate and inspect the database.

### Help & Information

| Command | Purpose | When to Use | Example |
|---------|---------|-------------|---------|
| `\?` | List all psql commands | You forget a backslash command | `\?` |
| `\h SQL_COMMAND` | Show SQL syntax help | You forget SQL syntax | `\h CREATE TABLE` |
| `\q` | Quit psql | Exit the session | `\q` |
| `\conninfo` | Show current connection details | Verify which DB you're connected to | `\conninfo` |

### Navigation

| Command | Purpose | When to Use | Example |
|---------|---------|-------------|---------|
| `\c database_name` | Connect to different database | Switch between databases | `\c test_db` |
| `\c database_name username` | Connect as different user | Test permissions | `\c production admin` |

### Schema Inspection (Critical for Learning SQL)

| Command | Output | Why Use It | When to Use | Example |
|---------|--------|------------|-------------|---------|
| `\l` | List all databases | See what databases exist | Starting work, forgot DB name | `\l` |
| `\dt` | List all tables in current schema | See what tables you can query | Before writing SELECT queries | `\dt` |
| `\dt+` | Tables with size & description | Check table sizes, find bloat | Debugging performance | `\dt+` |
| `\d table_name` | Table structure (columns, types) | **Most important: See column names/types before querying** | Before every INSERT/SELECT on unfamiliar table | `\d users` |
| `\d+ table_name` | Full table details + foreign keys | Understand relationships | Planning JOINs | `\d+ orders` |
| `\dn` | List schemas (namespaces) | Find tables in different schemas | Table "not found" errors | `\dn` |
| `\dv` | List views | See available views | Querying complex data | `\dv` |
| `\di` | List indexes | Check query optimization | Slow queries | `\di` |
| `\du` | List users/roles | Check permissions | Access errors | `\du` |
| `\df` | List functions | See available stored procedures | Before calling functions | `\df` |
| `\dx` | List extensions | Check installed extensions (e.g., uuid) | Using special data types | `\dx` |

**Pro Tip:** Always run `\d table_name` before writing queries for that table. It shows you the exact column names and types.

### Display Formatting

| Command | Effect | Why Use It | When to Use |
|---------|--------|------------|-------------|
| `\x on` | Expanded display (vertical) | Wide tables become readable | Tables with many columns |
| `\x off` | Table display (horizontal) | Return to normal grid | After viewing wide tables |
| `\x auto` | Auto-switch based on width | Best of both worlds | Set once at session start |
| `\timing on` | Show query execution time | Measure performance | Optimizing queries |
| `\timing off` | Hide execution time | Clean output | After optimization |

**Example:**
```sql
\x on
SELECT * FROM users WHERE id = 1;
-- Output:
-- id    | 1
-- name  | John Doe
-- email | john@example.com
-- ...much easier to read!
```

### Output Control

| Command | Purpose | When to Use | Example |
|---------|---------|-------------|---------|
| `\o filename.txt` | Redirect output to file | Save query results | `\o results.txt` |
| `\o` | Stop redirecting, return to screen | After saving results | `\o` |
| `\e` | Open last query in text editor | Edit complex queries | `\e` |
| `\ef function_name` | Edit function in editor | Modify stored procedures | `\ef calculate_total` |

### Data Import/Export

| Command | Purpose | When to Use | Example |
|---------|---------|-------------|---------|
| `\copy table TO 'file.csv' CSV HEADER` | Export table to CSV | Backup, analysis in Excel | `\copy users TO 'users.csv' CSV HEADER` |
| `\copy table FROM 'file.csv' CSV HEADER` | Import CSV to table | Seed test data | `\copy users FROM 'seed.csv' CSV HEADER` |
| `\copy (SELECT ...) TO 'file.csv' CSV` | Export query results | Custom data exports | `\copy (SELECT name, email FROM users WHERE active) TO 'active.csv' CSV HEADER` |

**Why \copy vs COPY:** `\copy` runs on your client machine (can access local files). `COPY` runs on the server (needs server file access/permissions).

### Monitoring & Debugging

| Command | Purpose | When to Use | Example |
|---------|---------|-------------|---------|
| `\watch 2` | Re-run last query every 2 seconds | Watch data change in real-time | After running `SELECT count(*) FROM jobs;` then `\watch 2` |
| `\echo message` | Print message | Debugging SQL scripts | `\echo 'Starting migration...'` |
| `\set` | Show all variables | Check session settings | `\set` |

---

## SQL Command Structure & Order

**Critical Concept:** SQL has a specific order for clauses. Understanding this order prevents syntax errors.

### SELECT Statement Order (Most Common Query)
```sql
SELECT columns
FROM table
JOIN other_table ON condition
WHERE filter_condition
GROUP BY columns
HAVING aggregate_condition
ORDER BY columns
LIMIT number
OFFSET number;
```

**Order is mandatory.** You cannot put `WHERE` after `GROUP BY`.

### INSERT Statement Order
```sql
INSERT INTO table (column1, column2)
VALUES (value1, value2);
```

### UPDATE Statement Order
```sql
UPDATE table
SET column1 = value1, column2 = value2
WHERE condition;
```

### DELETE Statement Order
```sql
DELETE FROM table
WHERE condition;
```

### CREATE TABLE Statement Order
```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    table_constraints
);
```

---

## Querying Data (SELECT)

### Basic SELECT
```sql
SELECT * FROM users;
```
**Why:** Retrieve all columns and rows from a table.
**When:** Exploring data, quick checks.
**Warning:** Avoid `SELECT *` in production (slow, returns unnecessary data).

### Select Specific Columns
```sql
SELECT name, email FROM users;
```
**Why:** Only get data you need (faster, clearer).
**When:** Always prefer this over `SELECT *`.

### WHERE Clause (Filtering)
```sql
SELECT name, email FROM users WHERE age > 18;
```
**Why:** Filter rows based on conditions.
**When:** You don't need all rows.

**Common operators:**
```sql
WHERE age = 25              -- Equality
WHERE age != 25             -- Not equal (also <>)
WHERE age > 18              -- Greater than
WHERE age >= 18             -- Greater or equal
WHERE age < 65              -- Less than
WHERE age BETWEEN 18 AND 65 -- Range (inclusive)
WHERE name LIKE 'John%'     -- Pattern matching (% = wildcard)
WHERE name ILIKE 'john%'    -- Case-insensitive LIKE
WHERE email IS NULL         -- Check for NULL
WHERE email IS NOT NULL     -- Check for not NULL
WHERE age IN (18, 21, 25)   -- Match any value in list
WHERE country = 'US' AND age > 18  -- AND logic
WHERE country = 'US' OR country = 'CA'  -- OR logic
WHERE NOT active            -- Negation
```

### ORDER BY (Sorting)
```sql
SELECT name FROM users ORDER BY age DESC;
```
**Why:** Sort results.
**When:** Displaying data to users, finding top/bottom values.

**Options:**
- `ASC` - Ascending (default, 1→10, A→Z)
- `DESC` - Descending (10→1, Z→A)

```sql
-- Multiple columns
SELECT name, age FROM users ORDER BY age DESC, name ASC;
-- First sort by age descending, then name ascending for ties
```

### LIMIT & OFFSET (Pagination)
```sql
SELECT * FROM users LIMIT 10;
```
**Why:** Restrict number of rows returned.
**When:** Pagination, testing queries on large tables.

```sql
SELECT * FROM users LIMIT 10 OFFSET 20;
-- Skip first 20 rows, return next 10 (page 3 if page size is 10)
```

### DISTINCT (Remove Duplicates)
```sql
SELECT DISTINCT country FROM users;
```
**Why:** Get unique values only.
**When:** Finding all possible values in a column.

### Aggregate Functions
```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(email) FROM users;  -- Doesn't count NULLs
SELECT AVG(age) FROM users;
SELECT SUM(price) FROM orders;
SELECT MAX(created_at) FROM logs;
SELECT MIN(age) FROM users;
```

**Why:** Compute values across rows.
**When:** Analytics, dashboards, summaries.

### GROUP BY (Aggregating by Category)
```sql
SELECT country, COUNT(*) FROM users GROUP BY country;
```
**Why:** Aggregate data per group.
**When:** "How many users per country?", "Total sales per product?"

**Rule:** Every column in SELECT must be either:
1. In the GROUP BY clause, OR
2. Inside an aggregate function (COUNT, SUM, etc.)

```sql
-- CORRECT
SELECT country, COUNT(*) FROM users GROUP BY country;

-- WRONG - age is not in GROUP BY and not aggregated
SELECT country, age, COUNT(*) FROM users GROUP BY country;

-- CORRECT - age is aggregated
SELECT country, AVG(age), COUNT(*) FROM users GROUP BY country;
```

### HAVING (Filtering After Aggregation)
```sql
SELECT country, COUNT(*) as user_count
FROM users
GROUP BY country
HAVING COUNT(*) > 100;
```

**Why:** Filter groups (WHERE filters rows before grouping, HAVING filters after).
**When:** "Show countries with more than 100 users", "Products with average rating > 4".

**WHERE vs HAVING:**
- `WHERE` filters individual rows before grouping
- `HAVING` filters groups after aggregation

```sql
-- Find countries with >100 adult users
SELECT country, COUNT(*) as count
FROM users
WHERE age >= 18           -- Filter rows first
GROUP BY country
HAVING COUNT(*) > 100;    -- Filter groups after
```

### JOIN (Combining Tables)

**Why JOIN:** In relational databases, data is split across tables to avoid duplication. JOINs reassemble it.

**Example schema:**
```sql
users:       orders:
id | name    id | user_id | total
1  | Alice   1  | 1       | 50
2  | Bob     2  | 1       | 75
             3  | 3       | 100
```

#### INNER JOIN (Most Common)
```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```
**Result:**
```
Alice | 50
Alice | 75
```
**Why:** Only returns rows where match exists in both tables.
**When:** Default choice for most queries.

#### LEFT JOIN (Include all from left table)
```sql
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```
**Result:**
```
Alice | 50
Alice | 75
Bob   | NULL
```
**Why:** Get all users even if they have no orders.
**When:** "Show all users and their orders (if any)".

#### RIGHT JOIN (Rarely used)
```sql
SELECT users.name, orders.total
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```
**Why:** Get all orders even if user was deleted.
**When:** Rarely needed; usually swap tables and use LEFT JOIN instead.

#### FULL OUTER JOIN (Rare)
```sql
SELECT users.name, orders.total
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;
```
**Why:** Get everything from both tables, with NULLs where no match.
**When:** Rarely needed in practice.

#### Multiple JOINs
```sql
SELECT users.name, orders.total, products.name
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN products ON orders.product_id = products.id;
```

**Pro Tip:** Use table aliases for readability:
```sql
SELECT u.name, o.total, p.name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN products p ON o.product_id = p.id;
```

### Subqueries
```sql
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);
```
**Why:** Use query results as input to another query.
**When:** Complex filtering, derived tables.

```sql
-- Scalar subquery (returns single value)
SELECT name FROM users WHERE age > (SELECT AVG(age) FROM users);

-- Table subquery (use as a table)
SELECT * FROM (
    SELECT name, age FROM users WHERE active = true
) AS active_users
WHERE age > 18;
```

### Common Table Expressions (CTEs) - WITH
```sql
WITH active_users AS (
    SELECT id, name FROM users WHERE active = true
)
SELECT name FROM active_users WHERE id > 100;
```

**Why:** Cleaner than nested subqueries, can be reused in same query.
**When:** Complex queries, recursive operations, readability.

```sql
-- Multiple CTEs
WITH
  adult_users AS (SELECT id, name FROM users WHERE age >= 18),
  big_orders AS (SELECT user_id, total FROM orders WHERE total > 1000)
SELECT u.name, o.total
FROM adult_users u
JOIN big_orders o ON u.id = o.user_id;
```

---

## Inserting Data (INSERT)

### Basic Insert
```sql
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 30);
```

**Syntax Order:**
1. `INSERT INTO table_name`
2. `(column1, column2, ...)`  ← Optional but recommended
3. `VALUES (value1, value2, ...)`

**Why specify columns:** Makes query resilient to schema changes, clearer intent.

### Insert Multiple Rows
```sql
INSERT INTO users (name, email, age) VALUES
    ('Alice', 'alice@example.com', 30),
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35);
```
**Why:** More efficient than separate INSERTs.
**When:** Seeding data, batch imports.

### Insert with Defaults
```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

INSERT INTO posts (title) VALUES ('Hello World');
-- id auto-generated, created_at uses default
```

### Insert and Return Values
```sql
INSERT INTO users (name, email)
VALUES ('Alice', 'alice@example.com')
RETURNING id, created_at;
```
**Why:** Get auto-generated IDs or default values immediately.
**When:** You need the generated ID for subsequent queries.

### Insert from SELECT
```sql
INSERT INTO archived_users (name, email)
SELECT name, email FROM users WHERE last_login < '2020-01-01';
```
**Why:** Copy data between tables.
**When:** Archiving, data migration.

### Upsert (INSERT ... ON CONFLICT)
```sql
INSERT INTO users (id, name, email)
VALUES (1, 'Alice', 'alice@example.com')
ON CONFLICT (id)
DO UPDATE SET email = EXCLUDED.email;
```
**Why:** Insert if doesn't exist, update if exists.
**When:** Idempotent operations, syncing data.

---

## Updating Data (UPDATE)

### Basic Update
```sql
UPDATE users
SET email = 'newemail@example.com'
WHERE id = 1;
```

**Syntax Order:**
1. `UPDATE table_name`
2. `SET column = value`
3. `WHERE condition`  ← **CRITICAL: Without WHERE, updates ALL rows!**

### Update Multiple Columns
```sql
UPDATE users
SET
    email = 'new@example.com',
    updated_at = NOW(),
    age = age + 1
WHERE id = 1;
```

### Update with Calculations
```sql
UPDATE products
SET price = price * 1.1  -- 10% increase
WHERE category = 'electronics';
```

### Update from Another Table
```sql
UPDATE orders o
SET total = oi.sum_total
FROM (
    SELECT order_id, SUM(price * quantity) as sum_total
    FROM order_items
    GROUP BY order_id
) oi
WHERE o.id = oi.order_id;
```

### Update and Return
```sql
UPDATE users
SET last_login = NOW()
WHERE id = 1
RETURNING id, name, last_login;
```

---

## Deleting Data (DELETE)

### Basic Delete
```sql
DELETE FROM users WHERE id = 1;
```

**Syntax Order:**
1. `DELETE FROM table_name`
2. `WHERE condition`  ← **CRITICAL: Without WHERE, deletes ALL rows!**

### Delete with Subquery
```sql
DELETE FROM users
WHERE id IN (
    SELECT user_id FROM banned_users
);
```

### Delete and Return
```sql
DELETE FROM users
WHERE last_login < '2020-01-01'
RETURNING id, name, email;
```

### TRUNCATE (Fast Delete All)
```sql
TRUNCATE TABLE users;
```
**Why:** Much faster than `DELETE FROM users` (doesn't log individual row deletions).
**When:** Clearing test data, resetting tables.
**Warning:** Cannot be rolled back in all cases, cannot use WHERE clause.

```sql
TRUNCATE TABLE users RESTART IDENTITY;
-- Also resets auto-increment counters (SERIAL)
```

### TRUNCATE with Foreign Keys
```sql
TRUNCATE TABLE users CASCADE;
```
**Why:** Also truncates tables that reference this table via foreign keys.
**Warning:** Dangerous! Cascades to related tables.

---

## Schema Creation (CREATE TABLE)

### Basic Table
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE,
    age INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Syntax Order:**
1. `CREATE TABLE table_name (`
2. Column definitions: `column_name datatype constraints`
3. Table constraints
4. `)`

### Common Data Types

| Type | Purpose | Example |
|------|---------|---------|
| `INTEGER` / `INT` | Whole numbers | `age INTEGER` |
| `BIGINT` | Large numbers | `views BIGINT` |
| `SERIAL` | Auto-incrementing integer | `id SERIAL` |
| `BIGSERIAL` | Auto-incrementing bigint | `id BIGSERIAL` |
| `NUMERIC(p,s)` | Exact decimal | `price NUMERIC(10,2)` (10 digits, 2 after decimal) |
| `REAL` / `DOUBLE PRECISION` | Floating point | `latitude REAL` |
| `TEXT` | Unlimited text | `description TEXT` |
| `VARCHAR(n)` | Text up to n chars | `username VARCHAR(50)` |
| `CHAR(n)` | Fixed-length text | `country_code CHAR(2)` |
| `BOOLEAN` | true/false | `active BOOLEAN` |
| `DATE` | Date only | `birth_date DATE` |
| `TIME` | Time only | `start_time TIME` |
| `TIMESTAMP` | Date + time | `created_at TIMESTAMP` |
| `TIMESTAMPTZ` | Timestamp with timezone | `created_at TIMESTAMPTZ` |
| `JSON` / `JSONB` | JSON data | `metadata JSONB` |
| `UUID` | Unique identifier | `id UUID` |
| `ARRAY` | Arrays | `tags TEXT[]` |

**SERIAL vs manual ID:**
```sql
-- SERIAL (automatic)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- PostgreSQL manages this
    name TEXT
);

-- Manual UUID
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT
);
```

### Column Constraints

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,                    -- Cannot be NULL
    sku TEXT UNIQUE,                       -- Must be unique
    price NUMERIC(10,2) CHECK (price > 0), -- Custom validation
    category TEXT DEFAULT 'general',       -- Default value
    created_at TIMESTAMP DEFAULT NOW()     -- Function default
);
```

**Constraint Types:**
- `NOT NULL` - Column must have a value
- `UNIQUE` - No two rows can have same value
- `PRIMARY KEY` - Combination of NOT NULL and UNIQUE, one per table
- `CHECK (condition)` - Custom validation rule
- `DEFAULT value` - Value if none provided
- `REFERENCES` - Foreign key (see below)

### Table Constraints (vs Column Constraints)

```sql
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id),  -- Composite primary key
    UNIQUE (order_id, product_id),       -- Composite unique
    CHECK (quantity > 0 AND quantity < 1000)
);
```

### IF NOT EXISTS
```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```
**Why:** Prevents error if table already exists.
**When:** Idempotent migrations, rerunnable scripts.

---

## Schema Modification (ALTER TABLE)

### Add Column
```sql
ALTER TABLE users ADD COLUMN phone TEXT;
```

### Add Column with Constraints
```sql
ALTER TABLE users
ADD COLUMN verified BOOLEAN NOT NULL DEFAULT false;
```

### Drop Column
```sql
ALTER TABLE users DROP COLUMN phone;
```

### Rename Column
```sql
ALTER TABLE users
RENAME COLUMN email TO email_address;
```

### Change Data Type
```sql
ALTER TABLE users
ALTER COLUMN age TYPE BIGINT;
```

### Add Constraint
```sql
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);
```

### Drop Constraint
```sql
ALTER TABLE users
DROP CONSTRAINT unique_email;
```

### Set Default
```sql
ALTER TABLE users
ALTER COLUMN created_at SET DEFAULT NOW();
```

### Drop Default
```sql
ALTER TABLE users
ALTER COLUMN created_at DROP DEFAULT;
```

### Set NOT NULL
```sql
ALTER TABLE users
ALTER COLUMN email SET NOT NULL;
```

### Drop NOT NULL
```sql
ALTER TABLE users
ALTER COLUMN email DROP NOT NULL;
```

---

## Constraints & Relationships

### Primary Keys

```sql
-- Single column
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

**Why:** Uniquely identifies each row.
**When:** Every table should have one (best practice).

### Foreign Keys

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total NUMERIC(10,2)
);
```

**Why:** Maintains referential integrity (can't have order for non-existent user).
**When:** Representing relationships between tables.

**Advanced foreign key options:**
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)
        ON DELETE CASCADE        -- Delete orders when user deleted
        ON UPDATE CASCADE,       -- Update orders.user_id if users.id changes
    total NUMERIC(10,2)
);
```

**ON DELETE options:**
- `CASCADE` - Delete child rows when parent deleted
- `SET NULL` - Set foreign key to NULL when parent deleted
- `RESTRICT` - Prevent deletion if children exist (default)
- `NO ACTION` - Same as RESTRICT

**ON UPDATE options:** Same as ON DELETE but for updates.

### UNIQUE Constraints

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE,
    username TEXT,
    UNIQUE (email, username)  -- Combination must be unique
);
```

### CHECK Constraints

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC(10,2) CHECK (price >= 0),
    stock INTEGER CHECK (stock >= 0),
    discount NUMERIC(3,2) CHECK (discount >= 0 AND discount <= 1),
    category TEXT CHECK (category IN ('electronics', 'books', 'clothing'))
);
```

**Why:** Enforce business rules at database level.
**When:** Data validation that should always be true.

---

## Indexes

### Why Indexes
Without index: Database scans every row (slow on large tables).
With index: Database uses index structure to find rows quickly (like a book's index).

**Trade-off:** Faster SELECTs, slower INSERTs/UPDATEs (index must be maintained).

### Basic Index
```sql
CREATE INDEX idx_users_email ON users(email);
```

**When:** Column frequently used in WHERE clauses or JOINs.

### Unique Index
```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```
**Why:** Like UNIQUE constraint, but can be partial or have custom expressions.

### Composite Index
```sql
CREATE INDEX idx_users_country_age ON users(country, age);
```
**Why:** Queries filtering by both columns benefit.
**Important:** Order matters! This index helps:
- `WHERE country = 'US' AND age > 18` ✓
- `WHERE country = 'US'` ✓
- `WHERE age > 18` ✗ (age is not first column)

### Partial Index
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```
**Why:** Smaller, faster index for subset of data.
**When:** Queries often filter by same condition.

### Drop Index
```sql
DROP INDEX idx_users_email;
```

### List Indexes
```sql
\di
-- or
SELECT * FROM pg_indexes WHERE tablename = 'users';
```

### When to Add Indexes

Add index if column is used in:
- `WHERE` clauses frequently
- `JOIN` conditions
- `ORDER BY` frequently
- Foreign key columns (automatically indexed by some databases, not PostgreSQL)

Don't index:
- Small tables (< 1000 rows)
- Columns with very low cardinality (e.g., boolean with 50/50 distribution)
- Columns rarely used in queries

---

## Transactions

### Why Transactions
**Problem:** Multiple operations that must all succeed or all fail together.

**Example:** Transferring money between accounts:
```sql
-- BAD: If crash happens between these, money disappears!
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
```

**Solution: Transaction:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Both or neither
```

### Basic Transaction
```sql
BEGIN;
-- your queries here
COMMIT;  -- Save changes
```

### Rollback Transaction
```sql
BEGIN;
UPDATE users SET role = 'admin' WHERE id = 1;
SELECT * FROM users WHERE id = 1;  -- Check if it looks right
ROLLBACK;  -- Undo everything
```

**Why:** Test changes before committing, undo mistakes.
**When:** Manual data fixes, testing queries.

### Savepoints (Partial Rollback)
```sql
BEGIN;
UPDATE users SET active = false WHERE id = 1;
SAVEPOINT my_savepoint;
UPDATE users SET active = false WHERE id = 2;
ROLLBACK TO SAVEPOINT my_savepoint;  -- Undo only second update
COMMIT;  -- Only first update is saved
```

### Transaction Isolation Levels
```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- queries
COMMIT;
```

**Levels:**
- `READ UNCOMMITTED` - Can read uncommitted changes (rarely used)
- `READ COMMITTED` - Default, can't read uncommitted changes
- `REPEATABLE READ` - Same query returns same results within transaction
- `SERIALIZABLE` - Strictest, transactions appear to run sequentially

**When:** Usually stick with default. Use SERIALIZABLE for critical financial operations.

---

## Common Patterns & Anti-Patterns

### Pattern: Check Before You Query
```bash
# Always check what exists first
\dt                    # List tables
\d table_name          # See structure
SELECT * FROM table_name LIMIT 5;  # Sample data
```

### Pattern: Test Queries with LIMIT
```sql
-- Start small
SELECT * FROM huge_table LIMIT 10;

-- Then remove LIMIT
SELECT * FROM huge_table;
```

### Pattern: Use EXPLAIN for Slow Queries
```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
-- Shows query plan (seq scan vs index scan)

EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
-- Actually runs query and shows timing
```

### Pattern: Safe Production Changes
```sql
-- Always use transactions for manual changes
BEGIN;
UPDATE users SET role = 'admin' WHERE email = 'boss@company.com';
SELECT * FROM users WHERE role = 'admin';  -- Verify
-- If good: COMMIT
-- If bad: ROLLBACK
```

### Anti-Pattern: SELECT * in Production
```sql
-- BAD
SELECT * FROM users;

-- GOOD
SELECT id, name, email FROM users;
```
**Why:** Wastes bandwidth, breaks if columns change, slower.

### Anti-Pattern: Missing WHERE in UPDATE/DELETE
```sql
-- DISASTER - Updates every row!
UPDATE users SET role = 'admin';

-- CORRECT
UPDATE users SET role = 'admin' WHERE id = 1;
```

### Anti-Pattern: N+1 Queries
```sql
-- BAD: Query in a loop (if you could avoid it)
-- For each user, fetch their orders separately
SELECT * FROM users;
-- Then for each user_id: SELECT * FROM orders WHERE user_id = ?

-- GOOD: Single query with JOIN
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### Anti-Pattern: Not Using Indexes
```sql
-- Slow on large tables
SELECT * FROM users WHERE email = 'test@example.com';

-- Add index
CREATE INDEX idx_users_email ON users(email);

-- Now fast
SELECT * FROM users WHERE email = 'test@example.com';
```

### Pattern: Finding Duplicate Data
```sql
-- Find duplicate emails
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### Pattern: Debugging Foreign Key Issues
```sql
-- Find orphaned records (orders with no user)
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

### Pattern: Resetting Test Data
```sql
-- Fast way to clear and reset
TRUNCATE TABLE users RESTART IDENTITY CASCADE;
-- Then insert test data
\copy users FROM 'test_data.csv' CSV HEADER;
```

### Pattern: Checking Query Performance
```sql
\timing on
SELECT COUNT(*) FROM large_table WHERE created_at > '2024-01-01';
-- Time: 4523.123 ms

-- Add index and test again
CREATE INDEX idx_large_table_created_at ON large_table(created_at);
SELECT COUNT(*) FROM large_table WHERE created_at > '2024-01-01';
-- Time: 12.456 ms  (much faster!)
```

---

## Quick Reference: SQL Statement Order

### SELECT (Full Syntax)
```sql
SELECT [DISTINCT] columns
FROM table
[JOIN table ON condition]
[WHERE condition]
[GROUP BY columns]
[HAVING condition]
[ORDER BY columns [ASC|DESC]]
[LIMIT number]
[OFFSET number];
```

### INSERT
```sql
INSERT INTO table (columns)
VALUES (values)
[RETURNING columns];
```

### UPDATE
```sql
UPDATE table
SET column = value
[WHERE condition]
[RETURNING columns];
```

### DELETE
```sql
DELETE FROM table
[WHERE condition]
[RETURNING columns];
```

### CREATE TABLE
```sql
CREATE TABLE [IF NOT EXISTS] table (
    column datatype [constraints],
    [table_constraints]
);
```

---

## Pro Tips for SQL Beginners

1. **Always use \d table_name before querying** - Saves you from typos and shows exact column names.

2. **Start SELECT queries without WHERE, then add filters** - See all data first, then narrow down.

3. **Use LIMIT liberally while learning** - Prevents accidentally printing millions of rows.

4. **Wrap manual changes in transactions** - `BEGIN;` your query, check results, then `COMMIT` or `ROLLBACK`.

5. **Keep \x auto on** - Automatically formats wide tables nicely.

6. **Check execution time with \timing on** - Understand query performance as you learn.

7. **Use meaningful aliases in JOINs** - `users u` is clearer than `users t1`.

8. **Read error messages carefully** - PostgreSQL error messages tell you exactly what's wrong and where.

9. **Don't fear EXPLAIN** - It's your friend for understanding why queries are slow.

10. **Build queries incrementally:**
    ```sql
    -- Step 1: Get basic data
    SELECT * FROM users;

    -- Step 2: Add filter
    SELECT * FROM users WHERE active = true;

    -- Step 3: Add join
    SELECT u.*, o.total
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.active = true;

    -- Step 4: Add aggregation
    SELECT u.name, COUNT(o.id) as order_count
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.active = true
    GROUP BY u.id, u.name;
    ```

---

## Emergency Commands

### Kill All Connections to Database
```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'database_name'
  AND pid <> pg_backend_pid();
```
**When:** Can't drop database, "database is being accessed by other users" error.

### Fix Sequence After Manual ID Insert
```sql
-- If you manually inserted id=1000, then auto-insert fails with duplicate key:
SELECT setval(pg_get_serial_sequence('table_name', 'id'),
              (SELECT MAX(id) FROM table_name));
```

### Drop Table with Dependencies
```sql
DROP TABLE table_name CASCADE;
```
**Warning:** Also drops views, foreign keys, etc. that depend on this table.

### View All Running Queries
```sql
SELECT pid, query, state, query_start
FROM pg_stat_activity
WHERE state = 'active';
```

### Cancel a Running Query
```sql
SELECT pg_cancel_backend(pid);  -- Graceful
-- or
SELECT pg_terminate_backend(pid);  -- Force kill
```

---

## Further Help

- **psql commands:** `\?`
- **SQL syntax:** `\h CREATE TABLE` (replace with any SQL command)
- **PostgreSQL docs:** https://www.postgresql.org/docs/
- **Interactive tutorial:** https://www.postgresqltutorial.com/

---

*This cheatsheet focuses on practical PostgreSQL/psql usage for developers learning SQL. For production deployments, also study performance tuning, security, backups, and replication.*
