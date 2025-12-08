# PostgreSQL (psql) Cheatsheet for Local Testing

A quick reference for developers and QA engineers performing local database testing, debugging, and data verification.

## 🔌 Connection & Basics

| Command | Description |
| :--- | :--- |
| `psql` | Enter the shell (uses current system user/db). |
| `psql -U <user> -d <db_name>` | Connect to a specific database with a specific user. |
| `psql -h localhost -p 5432` | Connect specifying host and port. |
| `\q` | **Quit/Exit** psql. |
| `\?` | Show help for psql backslash commands. |
| `\h <command>` | Show SQL syntax help (e.g., `\h CREATE TABLE`). |
| `\c <db_name>` | **Connect** (switch) to a different database. |
| `\conninfo` | Show details about current connection. |

---

## 🔍 Inspecting Schema (The "What exists?" Phase)

Essential for verifying that your migrations ran correctly.

| Command | Description |
| :--- | :--- |
| `\l` | **List** all databases on the server. |
| `\dt` | List all **tables** in the current schema. |
| `\dt+` | List tables with extra info (size, description). |
| `\d <table_name>` | **Describe** a specific table (columns, types, modifiers). |
| `\d+ <table_name>` | Describe table including **Foreign Keys** and storage details. |
| `\dn` | List schemas (namespaces). |
| `\dv` | List views. |
| `\du` | List users (roles) and their permissions. |
| `\di` | List indexes. |

> **Pro Tip:** If you can't find a table, checking the search path or listing schemas (`\dn`) is usually the first step.

---

## 🛠 Data Manipulation & Testing

Commands used to seed data, verify writes, or clean up after tests.

### Resetting Data
```sql
-- Fast delete of all rows. Resets identity (auto-increment) to 1.
TRUNCATE TABLE table_name RESTART IDENTITY;

-- Delete with logic (slower)
DELETE FROM table_name WHERE id > 10;
Importing/Exporting Test DataSQL-- Export query results to a CSV (Client side)
\copy (SELECT * FROM users) TO 'test_users.csv' WITH CSV HEADER;

-- Import data from CSV (Great for seeding)
\copy users FROM 'test_users.csv' WITH CSV HEADER;
Safe Manual Testing (Transactions)Don't mess up your local setup; use transactions to "preview" changes.SQLBEGIN;
UPDATE users SET role = 'admin' WHERE id = 1;
SELECT * FROM users; -- Check if it looks right
ROLLBACK; -- Undo everything
-- OR
COMMIT; -- Save it if it worked
🐛 Debugging & formattingWhen things go wrong or look messy in the terminal.The "Expanded Display" (Crucial for wide tables)If your table rows wrap around the screen and look unreadable:SQL\x on
-- Output becomes Key-Value pairs instead of a table grid.
-- Run query...
\x off
-- Back to normal table grid.
Performance ChecksWhy is the local query slow?SQL\timing on
-- Shows how long every query takes in milliseconds.

EXPLAIN ANALYZE SELECT * FROM large_table;
-- Runs the query and shows the execution plan + actual timing.
Watching a QueryRun a query repeatedly every 2 seconds (good for watching background jobs process data).SQLSELECT count(*) FROM jobs WHERE status = 'processing';
\watch 2
📝 Common SQL Snippets for TestingCheck Constraints & DefaultsSQLSELECT column_name, column_default, is_nullable
FROM information_schema.columns
WHERE table_name = 'your_table';
Find Broken SequencesIf your inserts are failing with "Duplicate Key" after a manual import:SQL-- Fix the sequence to match the max ID in the table
SELECT setval(pg_get_serial_sequence('my_table', 'id'), coalesce(max(id)+1, 1), false) FROM my_table;
Kill Stuck ConnectionsIf your test suite hangs or you can't drop a database because "it is being accessed":SQLSELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'target_db_name'
AND pid <> pg_backend_pid();
⚡ Output ManagementCommandDescription\o output.txtSend all following query results to output.txt.\oStop sending results to file (return to screen).\eOpen the last query in your default text editor (Vim/Nano).

1. The Safe Command (Best Practice)
This command deletes the table but prevents an error message if the table doesn't actually exist.

SQL

DROP TABLE IF EXISTS table_name;
2. The Standard Command
Use this if you are sure the table exists. It will throw an error if the table is missing.

SQL

DROP TABLE table_name;
3. The "Force" Command (Cascade)
If you try to delete a table and get an error saying "cannot drop table because other objects depend on it" (like Views or Foreign Keys), you need to use CASCADE.

Warning: This deletes the table AND any views or constraints linked to it.

SQL

DROP TABLE table_name CASCADE;
