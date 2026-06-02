# 📘 Day 5 — Indexes, Views, Stored Procedures & DB Performance

> **Focus:** Advanced database internals that separate junior candidates from senior ones in backend interviews.

---

## 📋 What You Will Learn

| Part | Topic | Key Concepts |
|------|-------|-------------|
| 1 | **Indexes** | B-tree internals, composite index, EXPLAIN, sargability |
| 2 | **Views** | Virtual tables, updatable views, materialized views |
| 3 | **Stored Procedures** | IN/OUT params, variables, control flow, SP vs Function |
| 4 | **Triggers** | BEFORE/AFTER, NEW/OLD, audit trail |
| 5 | **Transactions & ACID** | ACID deep-dive, isolation levels, deadlocks, SAVEPOINT |
| 6 | **Constraints** | PK, FK, UNIQUE, CHECK, ON DELETE options |
| 7 | **Query Optimization** | Sargability, EXPLAIN output, N+1, covering indexes |
| 8 | **Interview Q&A** | 20 theory questions with full answers |

---

## 📑 Index of Interview Questions

| Q# | Topic | Question |
|----|-------|----------|
| Q1 | Index | Clustered vs non-clustered index |
| Q2 | Index | How does a composite index work? Left-prefix rule |
| Q3 | Index | When does MySQL NOT use an index? |
| Q4 | Index | What is a covering index? |
| Q5 | Index | Too many indexes — what is the downside? |
| Q6 | View | View vs Materialized View |
| Q7 | View | Can you INSERT/UPDATE through a view? |
| Q8 | Stored Procedure | Stored Procedure vs Function |
| Q9 | Stored Procedure | Stored Procedure vs direct query — when to use? |
| Q10 | Trigger | BEFORE vs AFTER trigger — when to use each? |
| Q11 | Trigger | What are NEW and OLD in a trigger? |
| Q12 | Transaction | Explain ACID with a real example |
| Q13 | Transaction | What is a deadlock and how do you prevent it? |
| Q14 | Transaction | Isolation levels and anomalies |
| Q15 | Transaction | What is SAVEPOINT? |
| Q16 | Constraint | ON DELETE CASCADE vs SET NULL vs RESTRICT |
| Q17 | Constraint | UNIQUE vs PRIMARY KEY |
| Q18 | Optimization | What is query sargability? |
| Q19 | Optimization | How do you read EXPLAIN output? |
| Q20 | Optimization | What is the N+1 query problem? |

---

## 🧱 Sample Dataset

```sql
CREATE TABLE departments (
    id              INT PRIMARY KEY,
    department_name VARCHAR(50) NOT NULL
);

CREATE TABLE employees (
    id            INT PRIMARY KEY AUTO_INCREMENT,
    name          VARCHAR(50)  NOT NULL,
    department_id INT,
    salary        INT          NOT NULL,
    hire_date     DATE         NOT NULL,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

CREATE TABLE accounts (
    id      INT PRIMARY KEY,
    owner   VARCHAR(50),
    balance DECIMAL(10,2)
);

INSERT INTO departments VALUES (1,'Engineering'), (2,'HR'), (3,'Finance');

INSERT INTO employees VALUES
(1, 'Alice',   1, 80000, '2021-02-01'),
(2, 'Bob',     2, 60000, '2022-01-10'),
(3, 'Charlie', 1, 90000, '2020-05-12'),
(4, 'David',   3, 70000, '2023-04-01'),
(5, 'Eva',     2, 75000, '2021-07-15');

INSERT INTO accounts VALUES
(1, 'Alice', 10000.00),
(2, 'Bob',   5000.00);
```

**Employees at a glance:**

| id | name    | department_id | salary | hire_date  |
| --: | ------- | ------------: | -----: | ---------- |
| 1  | Alice   | 1             | 80,000 | 2021-02-01 |
| 2  | Bob     | 2             | 60,000 | 2022-01-10 |
| 3  | Charlie | 1             | 90,000 | 2020-05-12 |
| 4  | David   | 3             | 70,000 | 2023-04-01 |
| 5  | Eva     | 2             | 75,000 | 2021-07-15 |

---

---

# 🚀 Part 1 — Indexes

---

## 📖 Theory: What is an Index?

An **index** is a data structure maintained by the database engine that allows rows to be found quickly without scanning every row in the table (a *full table scan*).

**Analogy:** A book's index lets you jump directly to page 47 for "binary search" instead of reading every page.

Without an index, a query like `WHERE salary > 70000` reads all 5 rows and checks each one. With an index on `salary`, MySQL jumps directly to the relevant entries.

### How InnoDB Stores Indexes — B+ Tree

MySQL InnoDB uses a **B+ tree** for almost all indexes:

```
                        [70k | 80k]                ← internal node (keys only)
                       /      |      \
               [60k|70k]  [75k|80k]  [90k]         ← leaf nodes
                  ↓           ↓          ↓
              row data     row data   row data       ← actual data (clustered) or PK pointer (secondary)

Leaf nodes are linked: [60k|70k] ↔ [75k|80k] ↔ [90k]
This linked list makes RANGE queries fast.
```

### Types of Indexes

| Type | Description | Notes |
|------|-------------|-------|
| **Primary (Clustered)** | One per table; table rows physically stored in PK order | InnoDB always has one; `id` is typical |
| **Unique** | No duplicate values; one `NULL` allowed | Creates a constraint + index |
| **Regular (Non-clustered)** | Speeds up lookups; duplicates allowed | Most common manually created index |
| **Composite** | Covers multiple columns | Order matters — left-prefix rule applies |
| **Full-Text** | For keyword/text search | Use `MATCH ... AGAINST` instead of `LIKE` |
| **Covering** | Contains all columns needed by the query | Best — zero table access needed |

### When MySQL Uses an Index

- `WHERE` clause on the leading indexed column(s)
- `JOIN ON` condition on an indexed column
- `ORDER BY` on indexed column(s) matching index order
- `GROUP BY` on indexed column(s)

### When MySQL Does NOT Use an Index

| Situation | Example | Why |
|-----------|---------|-----|
| Function on indexed column | `WHERE YEAR(hire_date) = 2021` | Breaks sargability |
| Leading wildcard | `WHERE name LIKE '%Alice'` | Can't traverse B-tree from unknown start |
| OR with unindexed column | `WHERE salary > 70k OR name = 'Bob'` | One branch unindexed |
| Low cardinality column | `WHERE is_active = 1` (50% true) | Full scan is cheaper |
| Very small table | 5 rows | Optimizer skips index |
| Type mismatch | `WHERE id = '1'` (string vs int) | Implicit cast disables index |

---

## 📝 Index Examples

### Create a Simple Index

```sql
-- Speeds up: WHERE salary > 70000, ORDER BY salary
CREATE INDEX idx_salary
ON employees(salary);
```

### Create a Unique Index

```sql
-- Enforces uniqueness + speeds up lookups by email
CREATE UNIQUE INDEX idx_email
ON employees(email);
```

### Composite Index

```sql
-- Speeds up: WHERE department_id = 1 AND salary > 70000
-- Also works for: WHERE department_id = 1   (left prefix)
-- Does NOT help: WHERE salary > 70000        (skips leftmost column)
CREATE INDEX idx_dept_salary
ON employees(department_id, salary);
```

### Drop an Index

```sql
DROP INDEX idx_salary ON employees;
```

### Show All Indexes on a Table

```sql
SHOW INDEX FROM employees;
```

---

## 🔍 Reading EXPLAIN Output

`EXPLAIN` reveals how MySQL will execute a query — the most important query tuning tool.

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 70000;
```

**Sample output:**

| id | select_type | table     | type  | key        | rows | Extra       |
| -- | ----------- | --------- | ----- | ---------- | ---: | ----------- |
| 1  | SIMPLE      | employees | range | idx_salary |    2 | Using where |

### Key Fields to Focus On

| Field | What to Look For |
|-------|-----------------|
| **type** | `ALL` = full scan (bad) → `range` → `ref` → `eq_ref` → `const` (best) |
| **key** | Which index was chosen; `NULL` means no index used |
| **rows** | Estimated number of rows examined — lower is better |
| **Extra** | `Using index` = covering index (great); `Using filesort` = no index for ORDER BY (bad); `Using temporary` = temp table used (bad) |

### type Values Explained

```
ALL       → full table scan (every row examined)
index     → full index scan (all index entries, no table data)
range     → index range scan (between, >, <, IN)
ref       → index lookup by non-unique index
eq_ref    → index lookup, one row per join row (unique/PK join)
const     → single-row result from PRIMARY KEY or UNIQUE lookup
```

**Target:** `range` or better. `ALL` on large tables is a red flag.

```sql
-- Before index: type = ALL
EXPLAIN SELECT * FROM employees WHERE department_id = 1;

-- After adding index:
CREATE INDEX idx_dept ON employees(department_id);
EXPLAIN SELECT * FROM employees WHERE department_id = 1;
-- Now: type = ref, key = idx_dept
```

---

## ⚡ Query Sargability and Index Best Practices

```sql
-- ❌ NOT SARGABLE: function on indexed column (index ignored)
SELECT * FROM employees WHERE YEAR(hire_date) = 2021;

-- ✅ SARGABLE: range on column directly
SELECT * FROM employees WHERE hire_date BETWEEN '2021-01-01' AND '2021-12-31';

-- ❌ NOT SARGABLE: arithmetic on column
SELECT * FROM employees WHERE salary / 1000 > 70;

-- ✅ SARGABLE: arithmetic on the literal side
SELECT * FROM employees WHERE salary > 70 * 1000;

-- ❌ NOT SARGABLE: type mismatch (implicit cast)
SELECT * FROM employees WHERE id = '3';

-- ✅ SARGABLE: matching types
SELECT * FROM employees WHERE id = 3;
```

### Covering Index Example

```sql
-- Query only needs name and salary from employees
SELECT name, salary FROM employees WHERE department_id = 1;

-- Covering index: includes all columns the query touches
CREATE INDEX idx_covering ON employees(department_id, name, salary);

-- EXPLAIN Extra will show: "Using index" (no table row lookup needed)
```

---

---

# 🧩 Part 2 — Views

---

## 📖 Theory: What is a View?

A **view** is a named, saved SQL query stored on the server. It behaves like a virtual table — you can `SELECT` from it, and in some cases `INSERT`, `UPDATE`, and `DELETE` through it.

**Key fact:** A simple view stores **only the query definition**, not the data. Every time you query the view, MySQL re-runs the underlying query.

```
User query:  SELECT * FROM employee_dept_view WHERE dept = 'Engineering'
                           ↓
MySQL expands: SELECT e.name, d.department_name, e.salary
               FROM employees e JOIN departments d ON e.department_id = d.id
               WHERE d.department_name = 'Engineering'
```

### Why Use Views?

| Benefit | Example |
|---------|---------|
| **Simplicity** | Hide a complex 4-table JOIN behind a single name |
| **Security** | Grant SELECT on a view that excludes the `salary` column |
| **Reusability** | Multiple apps query the same logical shape |
| **Encapsulation** | Change the underlying tables without breaking app queries |

### View Limitations (MySQL)

- No indexes on a view directly (the underlying table indexes are still used)
- Cannot use `ORDER BY` in view definition reliably without `LIMIT`
- Updatable views have strict conditions (see Q7)

---

## 📝 View Examples

### Create a View

```sql
CREATE VIEW employee_dept_view AS
SELECT e.id, e.name, d.department_name, e.salary, e.hire_date
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

### Query the View

```sql
SELECT * FROM employee_dept_view;
```

**Result:**

| id | name    | department_name | salary | hire_date  |
| --: | ------- | --------------- | -----: | ---------- |
| 1  | Alice   | Engineering     | 80,000 | 2021-02-01 |
| 2  | Bob     | HR              | 60,000 | 2022-01-10 |
| 3  | Charlie | Engineering     | 90,000 | 2020-05-12 |
| 4  | David   | Finance         | 70,000 | 2023-04-01 |
| 5  | Eva     | HR              | 75,000 | 2021-07-15 |

```sql
-- Filter the view just like a table
SELECT name, salary
FROM employee_dept_view
WHERE department_name = 'Engineering';
```

| name    | salary |
| ------- | -----: |
| Alice   | 80,000 |
| Charlie | 90,000 |

### Security View — Hide Sensitive Columns

```sql
-- Public view: no salary column
CREATE VIEW employee_public_view AS
SELECT id, name, department_id, hire_date
FROM employees;

-- Grant SELECT on view, not on table
GRANT SELECT ON employee_public_view TO 'app_user'@'%';
```

### View with WITH CHECK OPTION

```sql
-- View: only HR employees
CREATE VIEW hr_employees AS
SELECT * FROM employees
WHERE department_id = 2
WITH CHECK OPTION;     -- prevents INSERT/UPDATE that would violate WHERE clause

-- This INSERT is rejected (department_id = 1 violates the view's WHERE)
INSERT INTO hr_employees (id, name, department_id, salary, hire_date)
VALUES (6, 'Frank', 1, 65000, '2024-01-01');
-- ERROR: CHECK OPTION failed

-- This INSERT succeeds (department_id = 2 satisfies the view's WHERE)
INSERT INTO hr_employees (id, name, department_id, salary, hire_date)
VALUES (6, 'Frank', 2, 65000, '2024-01-01');
```

### Modify and Drop a View

```sql
-- Replace view definition
CREATE OR REPLACE VIEW employee_dept_view AS
SELECT e.id, e.name, d.department_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 60000;

-- Drop view
DROP VIEW IF EXISTS employee_dept_view;

-- Show all views in current database
SHOW FULL TABLES WHERE TABLE_TYPE = 'VIEW';
```

---

---

# ⚙️ Part 3 — Stored Procedures

---

## 📖 Theory: What is a Stored Procedure?

A **stored procedure** is a pre-compiled, named block of SQL (and procedural logic) stored on the database server. It is called with `CALL` and can accept input parameters and return output values.

### Why Use Stored Procedures?

| Benefit | Detail |
|---------|--------|
| **Reduced network traffic** | One `CALL` replaces multiple round-trips from the app |
| **Security** | Grant `EXECUTE` without granting direct table access |
| **Performance** | Pre-parsed and partially optimized on first execution |
| **Encapsulation** | Business logic lives in one place, not scattered across apps |

### Parameter Modes

| Mode | Direction | Description |
|------|-----------|-------------|
| `IN` | Caller → Procedure | Read-only input; default mode |
| `OUT` | Procedure → Caller | Returns a value to the caller |
| `INOUT` | Both directions | Both passed in and modified |

---

## 📝 Stored Procedure Examples

### Basic Procedure: Salary Increase for a Department

```sql
DELIMITER //

CREATE PROCEDURE increase_salary(
    IN  dept_id          INT,
    IN  increment_amount INT
)
BEGIN
    UPDATE employees
    SET salary = salary + increment_amount
    WHERE department_id = dept_id;

    SELECT CONCAT('Updated ', ROW_COUNT(), ' employee(s)') AS message;
END //

DELIMITER ;
```

```sql
-- Execute
CALL increase_salary(1, 5000);
-- Result: "Updated 2 employee(s)" (Alice and Charlie are in dept 1)
```

### Procedure with OUT Parameter: Return a Value

```sql
DELIMITER //

CREATE PROCEDURE get_dept_max_salary(
    IN  dept_id    INT,
    OUT max_salary INT
)
BEGIN
    SELECT MAX(salary)
    INTO max_salary
    FROM employees
    WHERE department_id = dept_id;
END //

DELIMITER ;
```

```sql
-- Call and retrieve the OUT parameter
CALL get_dept_max_salary(1, @result);
SELECT @result AS max_salary_in_engineering;
-- Result: 90000 (Charlie's salary after the increase in previous example: actually 95000 now, but base = 90000)
```

### Procedure with IF / ELSEIF: Conditional Logic

```sql
DELIMITER //

CREATE PROCEDURE categorize_salary(
    IN  emp_id   INT,
    OUT category VARCHAR(20)
)
BEGIN
    DECLARE emp_salary INT;

    SELECT salary INTO emp_salary
    FROM employees
    WHERE id = emp_id;

    IF emp_salary >= 85000 THEN
        SET category = 'High';
    ELSEIF emp_salary >= 70000 THEN
        SET category = 'Mid';
    ELSE
        SET category = 'Low';
    END IF;
END //

DELIMITER ;
```

```sql
CALL categorize_salary(3, @cat);  -- Charlie: salary = 90000
SELECT @cat;                       -- Result: 'High'

CALL categorize_salary(2, @cat);  -- Bob: salary = 60000
SELECT @cat;                       -- Result: 'Low'
```

### Procedure with WHILE Loop

```sql
DELIMITER //

CREATE PROCEDURE apply_yearly_raises(IN years INT)
BEGIN
    DECLARE counter INT DEFAULT 0;
    WHILE counter < years DO
        UPDATE employees SET salary = ROUND(salary * 1.05);  -- 5% raise per year
        SET counter = counter + 1;
    END WHILE;
    SELECT 'Done' AS status, years AS years_applied;
END //

DELIMITER ;
```

```sql
CALL apply_yearly_raises(3);  -- Apply 5% raise 3 times
```

### List and Drop Procedures

```sql
SHOW PROCEDURE STATUS WHERE Db = 'your_database';

DROP PROCEDURE IF EXISTS increase_salary;
```

---

---

# 🔄 Part 4 — Triggers

---

## 📖 Theory: What is a Trigger?

A **trigger** is a stored program that fires **automatically** when a specified DML event (`INSERT`, `UPDATE`, `DELETE`) occurs on a table.

### Trigger Types

| Timing | Event | When it fires |
|--------|-------|---------------|
| `BEFORE INSERT` | INSERT | Before the new row is inserted |
| `AFTER INSERT` | INSERT | After the new row is inserted |
| `BEFORE UPDATE` | UPDATE | Before the row is changed |
| `AFTER UPDATE` | UPDATE | After the row is changed |
| `BEFORE DELETE` | DELETE | Before the row is deleted |
| `AFTER DELETE` | DELETE | After the row is deleted |

### NEW and OLD Pseudo-Records

| | INSERT | UPDATE | DELETE |
|--|--------|--------|--------|
| `NEW.col` | ✅ new value | ✅ new value | ❌ not available |
| `OLD.col` | ❌ not available | ✅ old value | ✅ value being deleted |

### When to Use BEFORE vs AFTER

- **BEFORE** — use when you want to **modify or validate** the data before it is saved (e.g., auto-format a field, enforce a business rule).
- **AFTER** — use when you want to **react to** a committed change (e.g., write an audit log, update a summary table).

---

## 📝 Trigger Examples

### Setup: Audit Table

```sql
CREATE TABLE salary_audit (
    audit_id   INT AUTO_INCREMENT PRIMARY KEY,
    emp_id     INT,
    emp_name   VARCHAR(50),
    old_salary INT,
    new_salary INT,
    changed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(50) DEFAULT USER()
);
```

### AFTER UPDATE Trigger: Salary Audit Log

```sql
CREATE TRIGGER trg_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary <> OLD.salary THEN        -- only log when salary actually changed
        INSERT INTO salary_audit (emp_id, emp_name, old_salary, new_salary)
        VALUES (OLD.id, OLD.name, OLD.salary, NEW.salary);
    END IF;
END;
```

```sql
-- Fire the trigger
UPDATE employees SET salary = 95000 WHERE id = 3;

-- Check the audit log
SELECT * FROM salary_audit;
```

**salary_audit result:**

| audit_id | emp_id | emp_name | old_salary | new_salary | changed_at          |
| -------: | -----: | -------- | ---------: | ---------: | ------------------- |
| 1        | 3      | Charlie  |     90,000 |     95,000 | 2024-06-01 10:30:00 |

### BEFORE INSERT Trigger: Auto-Uppercase Name

```sql
CREATE TRIGGER trg_format_name
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    SET NEW.name = CONCAT(
        UPPER(LEFT(NEW.name, 1)),
        LOWER(SUBSTRING(NEW.name, 2))
    );
END;
```

```sql
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (6, 'frank', 2, 65000, '2024-01-01');

SELECT name FROM employees WHERE id = 6;
-- Result: 'Frank'  (trigger formatted it automatically)
```

### BEFORE DELETE Trigger: Prevent Deleting High Earners

```sql
CREATE TRIGGER trg_protect_high_earner
BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary > 85000 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete an employee earning above 85,000';
    END IF;
END;
```

```sql
DELETE FROM employees WHERE id = 3;  -- Charlie earns 95,000
-- ERROR 1644: Cannot delete an employee earning above 85,000
```

### Show and Drop Triggers

```sql
SHOW TRIGGERS FROM your_database;

DROP TRIGGER IF EXISTS trg_salary_audit;
```

---

---

# 🔐 Part 5 — Transactions & ACID

---

## 📖 Theory: ACID Properties (Deep Dive)

Every database transaction must satisfy four properties:

### A — Atomicity

**"All or nothing."** A transaction is treated as a single unit. If any statement fails, **all** previous statements in the transaction are rolled back.

```
Transfer ₹1,000 from Alice to Bob:
  Step 1: Deduct ₹1,000 from Alice   ✅
  Step 2: Add ₹1,000 to Bob          ❌ (server crash)
  
  Without atomicity: Alice lost ₹1,000; Bob never received it.
  With atomicity:    The whole transaction rolls back — both accounts unchanged.
```

### C — Consistency

**"Database moves from one valid state to another valid state."** All constraints, rules, and triggers must be satisfied before and after the transaction. A transaction cannot leave the database in a partially valid state.

```
Before: Alice = ₹10,000 + Bob = ₹5,000 = total ₹15,000
After:  Alice = ₹9,000  + Bob = ₹6,000 = total ₹15,000   ✅ consistent
```

### I — Isolation

**"Concurrent transactions behave as if they ran serially."** Changes made by one in-progress transaction are not visible to others until committed (level-dependent).

```
Two users booking the last concert ticket simultaneously:
  T1 reads: 1 seat available
  T2 reads: 1 seat available
  T1 books: 0 seats
  T2 books: -1 seats ← problem without isolation!
  
  With proper isolation: T2 sees 0 seats after T1 commits, and fails cleanly.
```

### D — Durability

**"Committed data survives failures."** Once `COMMIT` succeeds, the changes are written to non-volatile storage (via WAL / redo log). A server crash immediately after `COMMIT` will not lose the data.

---

## 📝 Transaction Examples

### Basic Transaction — Bank Transfer

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;  -- Debit Alice
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;  -- Credit Bob

-- Check before committing
SELECT * FROM accounts;

COMMIT;   -- Make permanent
-- or
-- ROLLBACK;  -- Undo everything back to START TRANSACTION
```

**Accounts before:**

| id | owner | balance |
| --: | ----- | ------: |
| 1  | Alice | 10,000 |
| 2  | Bob   |  5,000 |

**Accounts after COMMIT:**

| id | owner | balance |
| --: | ----- | ------: |
| 1  | Alice |  9,000 |
| 2  | Bob   |  6,000 |

### SAVEPOINT — Partial Rollback

```sql
START TRANSACTION;

UPDATE employees SET salary = 85000 WHERE id = 1;   -- Alice: 80k → 85k
SAVEPOINT after_alice;                               -- mark this point

UPDATE employees SET salary = 100000 WHERE id = 2;  -- Bob: 60k → 100k (mistake!)

ROLLBACK TO after_alice;    -- undo Bob's update only; Alice's update is still pending

COMMIT;                     -- commit Alice's change
```

### Error Handling Pattern

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;

-- Simulate a check
SELECT @check := balance FROM accounts WHERE id = 1;

IF @check < 0 THEN
    ROLLBACK;
ELSE
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;
    COMMIT;
END IF;
```

---

## 📊 Isolation Levels

### The Four Isolation Levels and Concurrency Anomalies

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|----------------|-----------|---------------------|-------------|
| `READ UNCOMMITTED` | ❌ Possible | ❌ Possible | ❌ Possible |
| `READ COMMITTED` | ✅ Prevented | ❌ Possible | ❌ Possible |
| `REPEATABLE READ` | ✅ Prevented | ✅ Prevented | ⚠️ Possible* |
| `SERIALIZABLE` | ✅ Prevented | ✅ Prevented | ✅ Prevented |

*InnoDB prevents phantom reads in REPEATABLE READ using **gap locks**.
MySQL default isolation level: **REPEATABLE READ**

### Anomaly Definitions

**Dirty Read** — Transaction A reads data written by Transaction B, which has not yet committed. If B rolls back, A has read invalid data.

```
T1: UPDATE accounts SET balance = 99999 WHERE id = 1  (not committed yet)
T2: SELECT balance FROM accounts WHERE id = 1  → reads 99999 (dirty!)
T1: ROLLBACK  → balance is still 10000
T2 acted on a value that never officially existed
```

**Non-Repeatable Read** — Transaction A reads the same row twice and gets different values because Transaction B committed an update between the two reads.

```
T1: SELECT salary FROM employees WHERE id = 1  → 80000
T2: UPDATE employees SET salary = 85000 WHERE id = 1; COMMIT;
T1: SELECT salary FROM employees WHERE id = 1  → 85000  (different!)
```

**Phantom Read** — Transaction A runs the same range query twice and gets different sets of rows because Transaction B inserted or deleted rows between the two queries.

```
T1: SELECT COUNT(*) FROM employees WHERE department_id = 1  → 2
T2: INSERT INTO employees VALUES (6, 'Frank', 1, 72000, '2024-01-01'); COMMIT;
T1: SELECT COUNT(*) FROM employees WHERE department_id = 1  → 3  (phantom row!)
```

### Set Isolation Level

```sql
-- For current session only
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- For all new connections (requires SUPER privilege)
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Check current level
SHOW VARIABLES LIKE 'transaction_isolation';
```

---

## 💀 Deadlocks

### What is a Deadlock?

A **deadlock** occurs when two (or more) transactions are each waiting for a lock held by the other, creating a circular wait that can never be resolved.

```
T1 holds lock on employees row 1, wants lock on employees row 2
T2 holds lock on employees row 2, wants lock on employees row 1
→ Both wait forever — deadlock!
```

InnoDB **automatically detects deadlocks** and rolls back the transaction with the least "weight" (fewest rows modified). The rolled-back transaction gets error `ERROR 1213: Deadlock found`.

### Deadlock Prevention Strategies

| Strategy | How |
|----------|-----|
| **Consistent lock order** | Always lock rows/tables in the same order across transactions |
| **Keep transactions short** | Acquire and release locks quickly; don't hold locks across user interaction |
| **Use lower isolation** | `READ COMMITTED` acquires fewer gap locks |
| **Index your FK columns** | Unindexed FK updates lock the entire parent table |
| **Retry on deadlock** | Application logic: catch error 1213 and retry the transaction |

```sql
-- Check last deadlock
SHOW ENGINE INNODB STATUS;   -- look for "LATEST DETECTED DEADLOCK" section
```

---

---

# 📌 Part 6 — Constraints

---

## 📖 Theory: What are Constraints?

**Constraints** are rules enforced on table columns (or the whole table) that preserve data integrity. They are checked automatically by the database on every `INSERT`, `UPDATE`, and `DELETE`.

---

## 📝 All Constraint Types

### PRIMARY KEY

```sql
-- Single column
CREATE TABLE employees (
    id INT PRIMARY KEY,
    ...
);

-- Composite primary key
CREATE TABLE order_items (
    order_id    INT,
    product_id  INT,
    quantity    INT,
    PRIMARY KEY (order_id, product_id)
);
```

- Implicitly `NOT NULL` + `UNIQUE`
- One per table
- Creates a **clustered index** in InnoDB

### FOREIGN KEY

```sql
CREATE TABLE employees (
    id            INT PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

**ON DELETE / ON UPDATE Options:**

| Option | Behavior |
|--------|---------|
| `CASCADE` | Automatically delete/update child rows |
| `SET NULL` | Set FK column to `NULL` in child rows |
| `RESTRICT` | Block the operation if child rows exist |
| `NO ACTION` | Same as `RESTRICT` in MySQL (default) |

```sql
-- CASCADE example: deleting a department removes its employees automatically
ALTER TABLE employees
    ADD CONSTRAINT fk_dept
    FOREIGN KEY (department_id) REFERENCES departments(id)
    ON DELETE CASCADE;

DELETE FROM departments WHERE id = 2;
-- Bob and Eva (dept 2) are automatically deleted from employees
```

### UNIQUE

```sql
-- Column-level
email VARCHAR(100) UNIQUE

-- Table-level (composite unique)
CONSTRAINT uq_name_dept UNIQUE (name, department_id)
```

- Allows **one** `NULL` value (NULL ≠ NULL in SQL standard)
- Creates a non-clustered unique index

### NOT NULL

```sql
name    VARCHAR(50) NOT NULL,
salary  INT         NOT NULL DEFAULT 0
```

### CHECK (MySQL 8.0.16+)

```sql
CREATE TABLE employees (
    id     INT PRIMARY KEY,
    salary INT CHECK (salary > 0),
    age    INT CHECK (age BETWEEN 18 AND 65)
);

-- Add CHECK to existing table
ALTER TABLE employees
    ADD CONSTRAINT chk_salary CHECK (salary > 0);
```

### DEFAULT

```sql
is_active  BOOLEAN  NOT NULL DEFAULT TRUE,
created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
country    VARCHAR(50) DEFAULT 'India'
```

### Add / Drop Constraints on Existing Table

```sql
-- Add a constraint
ALTER TABLE employees
    ADD CONSTRAINT chk_salary CHECK (salary >= 0);

-- Drop a named constraint
ALTER TABLE employees
    DROP CONSTRAINT chk_salary;

-- Drop a foreign key
ALTER TABLE employees
    DROP FOREIGN KEY fk_dept;
```

---

---

# ⚡ Part 7 — Query Optimization

---

## 📖 Theory: How to Write Fast Queries

### 1. Index Sargability

A query is **sargable** (Search-ARGument-able) if the WHERE clause can use an index to limit the rows scanned.

```sql
-- ❌ NOT sargable: function wraps the column → index bypassed
WHERE UPPER(name) = 'ALICE'
WHERE MONTH(hire_date) = 6
WHERE salary * 12 > 1000000

-- ✅ Sargable: comparison directly on column
WHERE name = 'Alice'
WHERE hire_date BETWEEN '2021-06-01' AND '2021-06-30'
WHERE salary > 83333
```

### 2. SELECT Only What You Need

```sql
-- ❌ Fetches every column — more I/O, more memory, prevents covering index
SELECT * FROM employees WHERE department_id = 1;

-- ✅ Only needed columns
SELECT id, name, salary FROM employees WHERE department_id = 1;
```

### 3. Use LIMIT on Large Tables

```sql
-- ❌ Returns millions of rows to the app
SELECT * FROM audit_log WHERE action = 'LOGIN';

-- ✅ Paginate
SELECT * FROM audit_log WHERE action = 'LOGIN'
ORDER BY created_at DESC LIMIT 20 OFFSET 0;
```

### 4. Avoid Leading Wildcards in LIKE

```sql
-- ❌ Can't use index — unknown starting character
WHERE name LIKE '%Alice%'

-- ✅ Can use index — fixed starting prefix
WHERE name LIKE 'Ali%'

-- For full-text search, use FULLTEXT index + MATCH AGAINST instead
```

### 5. Prefer JOINs over Correlated Subqueries

```sql
-- ❌ Correlated subquery: re-executes per row
SELECT name FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);

-- ✅ Pre-compute averages once, then join
SELECT e.name
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees GROUP BY department_id
) d ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

### 6. Avoid SELECT DISTINCT Unless Necessary

```sql
-- ❌ DISTINCT forces de-duplication — expensive sort/hash operation
SELECT DISTINCT department_id FROM employees;

-- ✅ Use EXISTS or GROUP BY if that's the intent
SELECT department_id FROM employees GROUP BY department_id;
```

### 7. Index Your JOIN Columns

```sql
-- The join column on the right side (departments.id) should be indexed
-- It already is (PRIMARY KEY). Ensure the left side is also indexed:
CREATE INDEX idx_emp_dept ON employees(department_id);

EXPLAIN
SELECT e.name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.id;
-- key should show: idx_emp_dept (not NULL)
```

### 8. The N+1 Query Problem

An N+1 problem is when you run 1 query to get N records, then run 1 more query per record — resulting in N+1 total queries.

```sql
-- ❌ N+1: Fetch all employees, then for each one fetch their department (N queries)
SELECT * FROM employees;                              -- 1 query → 5 rows
-- For row 1: SELECT * FROM departments WHERE id = 1  -- +1 query
-- For row 2: SELECT * FROM departments WHERE id = 2  -- +1 query
-- ...5 employees = 1 + 5 = 6 queries total

-- ✅ Fix: one JOIN fetches everything
SELECT e.name, d.department_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.id;        -- 1 query, 0 N+1
```

### EXPLAIN ANALYZE (MySQL 8.0+)

```sql
-- Shows actual execution times, not just estimates
EXPLAIN ANALYZE
SELECT e.name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 70000;
```

---

---

# 🎯 Part 8 — Top 20 Interview Q&A

---

### Q1 — What is the difference between a clustered and a non-clustered index?

**A:**

| | Clustered Index | Non-Clustered (Secondary) Index |
|--|-----------------|----------------------------------|
| Storage | Table rows physically stored in index order | Separate structure; leaf nodes store PK reference |
| Count | One per table | Many per table |
| In InnoDB | PRIMARY KEY is always clustered | All other indexes are non-clustered |
| Lookup speed | Direct — data is at the leaf | Two-step: index lookup → then PK lookup to fetch row |

```sql
-- Primary key = clustered index (implicit in InnoDB)
CREATE TABLE employees (id INT PRIMARY KEY, ...);

-- Secondary (non-clustered) index
CREATE INDEX idx_salary ON employees(salary);
-- Leaf nodes of this index store: (salary_value → id)
-- MySQL then fetches the row using that id via the clustered index
```

> **Interview answer:** "In InnoDB, the PRIMARY KEY is the clustered index — rows are physically stored in PK order. Every secondary index stores the PK value in its leaf nodes and requires a second lookup to fetch the full row. This is why choosing a small, sequential PK (like INT AUTO_INCREMENT) matters for performance."

---

### Q2 — How does a composite index work? What is the left-prefix rule?

**A:** A composite index on `(a, b, c)` is stored sorted first by `a`, then by `b` within the same `a`, then by `c`.

**Left-prefix rule:** MySQL can only use the index starting from the leftmost column.

```sql
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- ✅ Uses index (department_id is leftmost)
WHERE department_id = 1
WHERE department_id = 1 AND salary > 70000

-- ❌ Cannot use this composite index (skips leftmost column)
WHERE salary > 70000

-- ❌ Cannot use for range on first column + filter on second
WHERE department_id > 1 AND salary = 80000
-- (range on first col stops index use for subsequent cols)
```

> **Interview tip:** Column order in a composite index matters. Put equality-filter columns first, range-filter columns last.

---

### Q3 — When does MySQL choose NOT to use an index?

**A:** MySQL's query optimizer may skip an index when:

1. **Low cardinality** — column has few distinct values (e.g., boolean, status flag with 3 values). Full scan is cheaper when >15-20% of rows would be returned.
2. **Function on column** — `WHERE YEAR(hire_date) = 2021` — breaks sargability.
3. **Leading wildcard** — `WHERE name LIKE '%Smith'` — B-tree can't start from an unknown prefix.
4. **Type mismatch** — implicit casting disables the index.
5. **Very small table** — optimizer judges full scan is faster than index overhead.
6. **Outdated statistics** — run `ANALYZE TABLE employees` to refresh.

```sql
-- Force index use (override optimizer — use with caution)
SELECT * FROM employees FORCE INDEX (idx_salary) WHERE salary > 70000;

-- Ignore a specific index
SELECT * FROM employees IGNORE INDEX (idx_salary) WHERE salary > 70000;
```

---

### Q4 — What is a covering index?

**A:** A **covering index** is an index that contains **all the columns a query needs** — so MySQL can answer the query entirely from the index without touching the actual table rows.

```sql
-- Query: needs name, salary, department_id
SELECT name, salary FROM employees WHERE department_id = 1;

-- Covering index for this query:
CREATE INDEX idx_cover ON employees(department_id, name, salary);
-- Now MySQL reads only the index — no row lookup needed
```

**How to identify:** `EXPLAIN` shows `Extra: Using index` (not `Using where; Using index` — that means the index is used for filtering AND covering).

> **Performance impact:** A covering index eliminates the second lookup to the clustered index. For frequently-run queries on large tables, this can halve the I/O.

---

### Q5 — What is the downside of having too many indexes?

**A:**

| Impact | Detail |
|--------|--------|
| **Slower writes** | Every `INSERT`, `UPDATE`, `DELETE` must update all indexes on the table |
| **More storage** | Each index is a separate B-tree structure |
| **Optimizer confusion** | Too many choices can lead the optimizer to pick a suboptimal index |
| **Maintenance overhead** | Indexes become fragmented; periodic `OPTIMIZE TABLE` needed |

> **Rule of thumb:** Index columns used in `WHERE`, `JOIN ON`, `ORDER BY`, and `GROUP BY`. Don't index columns with very low cardinality. Remove indexes that are never used (`PERFORMANCE_SCHEMA` or slow query log can identify these).

---

### Q6 — What is the difference between a View and a Materialized View?

**A:**

| | View | Materialized View |
|--|------|-------------------|
| Data storage | No data stored; query re-runs each time | Query result physically stored as a table |
| Freshness | Always up to date | Can be stale; needs periodic refresh |
| Query speed | Depends on underlying query complexity | Fast — reads pre-computed data |
| Storage cost | Zero | Proportional to result set size |
| MySQL support | ✅ Native | ❌ Not native in MySQL 8.0 (emulate with scheduled procedure + table) |

```sql
-- Emulating a materialized view in MySQL:

-- 1. Create the summary table
CREATE TABLE dept_salary_summary AS
SELECT department_id, COUNT(*) AS headcount, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;

-- 2. Refresh it on a schedule (event or procedure)
CREATE EVENT refresh_summary
ON SCHEDULE EVERY 1 HOUR
DO
    REPLACE INTO dept_salary_summary
    SELECT department_id, COUNT(*), AVG(salary)
    FROM employees GROUP BY department_id;
```

---

### Q7 — Can you INSERT or UPDATE data through a view?

**A:** Yes, but only if the view is **updatable**. A view is updatable in MySQL when it satisfies all of these:

- Based on a single table (no JOINs in the view)
- No `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`
- No aggregate functions (`SUM`, `COUNT`, etc.)
- No subqueries in the `SELECT` list

```sql
-- ✅ Updatable view (single table, no aggregates)
CREATE VIEW hr_employees AS
SELECT id, name, salary FROM employees WHERE department_id = 2;

UPDATE hr_employees SET salary = 65000 WHERE id = 2;   -- works
INSERT INTO hr_employees (id, name, salary) VALUES (7, 'Grace', 62000);  -- works (dept_id defaults to NULL)

-- ❌ NOT updatable (JOIN)
CREATE VIEW employee_dept_view AS
SELECT e.name, d.department_name FROM employees e JOIN departments d ON e.department_id = d.id;

UPDATE employee_dept_view SET name = 'Bob2' WHERE name = 'Bob';  -- ERROR
```

---

### Q8 — What is the difference between a Stored Procedure and a Function?

**A:**

| | Stored Procedure | Function |
|--|-----------------|----------|
| Return value | None (or via `OUT` params) | Must return exactly one value |
| Use in SELECT | ❌ Cannot be called in SELECT | ✅ Can be used in expressions |
| Called with | `CALL proc_name(args)` | `SELECT func_name(args)` |
| Transaction control | ✅ Can use `COMMIT`, `ROLLBACK` | ❌ Cannot |
| Data modification | ✅ Can `INSERT`, `UPDATE`, `DELETE` | Limited (must be deterministic for replication) |
| Purpose | Multi-step business logic | Computation / transformation |

```sql
-- Function: used inline in SELECT
CREATE FUNCTION get_annual_salary(monthly INT)
RETURNS INT DETERMINISTIC
RETURN monthly * 12;

SELECT name, salary, get_annual_salary(salary) AS annual FROM employees;

-- Procedure: used with CALL
CALL increase_salary(1, 5000);
```

---

### Q9 — When would you use a Stored Procedure instead of writing queries in the application?

**A:** Use a stored procedure when:

1. **Multiple statements must run atomically** as part of a complex business operation.
2. **Security** — grant `EXECUTE` to an app user without granting direct `SELECT/UPDATE` on sensitive tables.
3. **Reduce network round-trips** — one `CALL` instead of 5 separate statements.
4. **Shared logic** — multiple apps (Java backend, Python scripts, reporting tools) share the same DB logic without duplication.

**When NOT to use stored procedures:**
- When the logic is complex enough to warrant unit testing (SPs are hard to test).
- When you need version control, code review, CI/CD pipelines (code in app is easier to manage).
- When the logic changes frequently (deployment requires DB migration).

---

### Q10 — When should you use a BEFORE trigger vs an AFTER trigger?

**A:**

| Use BEFORE | Use AFTER |
|-----------|---------|
| Modify `NEW` values before they're written | React to a committed change |
| Validate/reject the operation with `SIGNAL` | Write to audit/log tables |
| Auto-populate derived columns | Update a summary/aggregation table |
| Enforce custom business rules | Notify another system |

```sql
-- BEFORE: modify data before it's saved
CREATE TRIGGER trg_uppercase_name
BEFORE INSERT ON employees
FOR EACH ROW SET NEW.name = UPPER(NEW.name);

-- AFTER: log what was saved
CREATE TRIGGER trg_audit
AFTER UPDATE ON employees
FOR EACH ROW
INSERT INTO salary_audit VALUES (OLD.id, OLD.salary, NEW.salary, NOW());
```

> **BEFORE** = gate-keeper / formatter. **AFTER** = observer / logger.

---

### Q11 — What are NEW and OLD in a trigger, and when are they available?

**A:** `NEW` and `OLD` are special pseudo-records inside a trigger body that give access to the row data being affected.

| | INSERT | UPDATE | DELETE |
|--|--------|--------|--------|
| `OLD` | ❌ (no old row) | ✅ Value before update | ✅ Value being deleted |
| `NEW` | ✅ Value being inserted | ✅ Value after update | ❌ (no new row) |
| Can modify | `NEW.col = ...` | `NEW.col = ...` | Neither |

```sql
-- In AFTER UPDATE trigger:
IF NEW.salary > OLD.salary THEN
    -- salary increased
    INSERT INTO audit_log (message)
    VALUES (CONCAT(OLD.name, ' got a raise from ', OLD.salary, ' to ', NEW.salary));
END IF;
```

---

### Q12 — Explain ACID with a real-world example.

**A:** Consider a bank transfer of ₹1,000 from Alice to Bob:

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 1000 WHERE id = 1;  -- Alice: 10k → 9k
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;  -- Bob:   5k → 6k
COMMIT;
```

- **Atomicity** — if the server crashes after the debit but before the credit, the whole transaction rolls back. Alice's ₹1,000 is not lost.
- **Consistency** — total money in system is ₹15,000 before and after. FK constraints, CHECK constraints all remain satisfied.
- **Isolation** — if a concurrent report is running `SELECT SUM(balance)`, it sees either the old state (both original) or the new state (both updated) — never a half-done transfer.
- **Durability** — once `COMMIT` returns successfully, the transfer is written to the redo log. A crash one millisecond later cannot undo it.

---

### Q13 — What is a deadlock and how do you prevent it?

**A:** A deadlock occurs when two transactions hold locks the other needs:

```
T1: locks row A, waits for row B
T2: locks row B, waits for row A  → circular wait → deadlock
```

InnoDB detects this automatically and rolls back the smaller transaction (error 1213).

**Prevention strategies:**

1. **Consistent lock ordering** — always access tables and rows in the same order.
2. **Short transactions** — acquire, execute, release quickly. Never hold a lock across user input.
3. **Use `SELECT ... FOR UPDATE`** to acquire locks upfront when you know you'll update.
4. **Lower isolation level** — `READ COMMITTED` acquires fewer gap locks.
5. **Application-level retry** — catch error 1213 and re-run the transaction.

```sql
-- Diagnose last deadlock
SHOW ENGINE INNODB STATUS\G
-- Look for: "LATEST DETECTED DEADLOCK"
```

---

### Q14 — What are the four isolation levels and what anomaly does each prevent?

**A:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|:----------:|:-------------------:|:------------:|:-----------:|
| `READ UNCOMMITTED` | ❌ | ❌ | ❌ | Highest |
| `READ COMMITTED` | ✅ | ❌ | ❌ | High |
| `REPEATABLE READ` | ✅ | ✅ | ⚠️* | Medium |
| `SERIALIZABLE` | ✅ | ✅ | ✅ | Lowest |

*InnoDB prevents phantom reads in REPEATABLE READ using gap locks.

- **Dirty Read** — reading another transaction's uncommitted changes.
- **Non-Repeatable Read** — same row read twice gives different values.
- **Phantom Read** — same range query returns different row counts.

MySQL default: `REPEATABLE READ`. Most applications are fine with `READ COMMITTED`.

---

### Q15 — What is a SAVEPOINT and when would you use it?

**A:** A `SAVEPOINT` marks a point within a transaction to which you can roll back without undoing the entire transaction.

```sql
START TRANSACTION;

INSERT INTO employees VALUES (6, 'Frank', 2, 65000, '2024-01-01');
SAVEPOINT sp_after_frank;

INSERT INTO employees VALUES (7, 'Grace', 99, 70000, '2024-01-01');  -- dept 99 doesn't exist
-- FK violation or business rule error!

ROLLBACK TO sp_after_frank;    -- undo Grace's insert, keep Frank's

COMMIT;                        -- Frank is committed
```

**Use case:** Long batch operations where you want to undo only the most recent step if it fails, without losing all previous successful steps.

---

### Q16 — What is the difference between ON DELETE CASCADE, SET NULL, and RESTRICT?

**A:**

```sql
-- CASCADE: deleting a department deletes its employees too
FOREIGN KEY (department_id) REFERENCES departments(id) ON DELETE CASCADE;

-- SET NULL: deleting a department sets employees.department_id to NULL
FOREIGN KEY (department_id) REFERENCES departments(id) ON DELETE SET NULL;

-- RESTRICT (default): prevents deleting a department that still has employees
FOREIGN KEY (department_id) REFERENCES departments(id) ON DELETE RESTRICT;
```

| Option | What happens to child rows | Use when |
|--------|---------------------------|----------|
| `CASCADE` | Automatically deleted | Child rows meaningless without parent (e.g., order items without order) |
| `SET NULL` | FK column set to NULL | Child rows can exist without parent (e.g., employee after dept is dissolved) |
| `RESTRICT` | Operation blocked | Must explicitly reassign children before deleting parent |

---

### Q17 — What is the difference between UNIQUE constraint and PRIMARY KEY?

**A:**

| | PRIMARY KEY | UNIQUE |
|--|------------|--------|
| NULLs | Not allowed (implicit NOT NULL) | One NULL allowed |
| Count per table | Exactly one | Multiple |
| Index type | Clustered (InnoDB) | Non-clustered |
| Purpose | Row identity | Prevent duplicates in other columns |

```sql
CREATE TABLE employees (
    id    INT         PRIMARY KEY,       -- row identity, clustered
    email VARCHAR(100) UNIQUE NOT NULL,  -- no duplicate emails
    phone VARCHAR(15)  UNIQUE            -- no duplicate phones; one NULL allowed
);
```

---

### Q18 — What is query sargability and why does it matter?

**A:** A predicate is **sargable** if the database can use an index to evaluate it efficiently. Non-sargable predicates force a full table scan even when an index exists.

**The main culprit:** wrapping an indexed column in a function.

```sql
-- ❌ Non-sargable (function on column — index on hire_date is bypassed)
WHERE YEAR(hire_date) = 2021

-- ✅ Sargable rewrite
WHERE hire_date >= '2021-01-01' AND hire_date < '2022-01-01'

-- ❌ Non-sargable
WHERE LOWER(name) = 'alice'

-- ✅ Store consistently or use generated column + index
WHERE name = 'Alice'

-- ❌ Non-sargable
WHERE salary / 12 > 6000

-- ✅ Move math to the literal side
WHERE salary > 72000
```

---

### Q19 — How do you read EXPLAIN output to identify performance problems?

**A:** Focus on these three fields:

**`type` column — the access method:**
```
ALL       → ⚠️  full table scan — always investigate on large tables
index     → ⚠️  full index scan
range     → ✅  index range scan
ref       → ✅  non-unique index lookup
eq_ref    → ✅✅ unique/PK join (one row per join)
const     → ✅✅ single row (WHERE id = 5)
```

**`key` column:** `NULL` = no index used — add one.

**`Extra` column:**
```
Using filesort   → ORDER BY can't use an index — add index on ORDER BY column
Using temporary  → GROUP BY/ORDER BY uses a temp table — try to rewrite
Using index      → covering index — great, no table access needed
Using where      → filtering after index read — normal
```

```sql
-- Example: no index on department_id
EXPLAIN SELECT name FROM employees WHERE department_id = 1;
-- type: ALL, key: NULL, rows: 5  ← bad (small table, but scales badly)

-- After adding index
CREATE INDEX idx_dept ON employees(department_id);
EXPLAIN SELECT name FROM employees WHERE department_id = 1;
-- type: ref, key: idx_dept, rows: 2  ← good
```

---

### Q20 — What is the N+1 query problem and how do you fix it?

**A:** The N+1 problem occurs when code fetches N parent records, then executes one additional query for each child record — totalling N+1 database round-trips instead of 1.

**Example (Java/ORM pseudocode pattern):**
```
List<Employee> employees = db.query("SELECT * FROM employees");   // 1 query
for (Employee e : employees) {                                     // 5 iterations
    Department d = db.query("SELECT * FROM departments WHERE id = " + e.deptId);
    // 5 extra queries!
}
// Total: 6 queries for 5 employees
```

**SQL fix — use JOIN:**
```sql
-- 1 query replaces 6
SELECT e.name, d.department_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

**ORM fix:** Use eager loading (`JOIN FETCH` in JPA, `include` in ActiveRecord, `select_related` in Django ORM).

> **Interview answer:** "The N+1 problem typically appears in ORM-heavy applications and becomes critical at scale. The fix is always to batch the data fetching into a single JOIN query rather than looping and querying."

---

---

# 🧠 Key Takeaways

| Topic | Most Important Point |
|-------|---------------------|
| **Indexes** | Index columns in WHERE/JOIN/ORDER BY; avoid functions on indexed columns; use EXPLAIN |
| **Composite Index** | Left-prefix rule; order matters (equality columns first, range last) |
| **Views** | Virtual — no stored data; only simple single-table views are updatable |
| **Stored Procedures** | Use for multi-step logic, security, reducing round-trips; hard to unit test |
| **Triggers** | BEFORE = modify/validate; AFTER = log/react; use NEW and OLD carefully |
| **ACID** | Atomicity = all or nothing; Isolation level controls concurrency trade-offs |
| **Isolation Levels** | MySQL default = REPEATABLE READ; READ COMMITTED is fine for most apps |
| **Deadlocks** | Always lock in consistent order; keep transactions short; retry on error 1213 |
| **Constraints** | CASCADE = auto-delete children; SET NULL = orphan safely; RESTRICT = block |
| **Query Optimization** | EXPLAIN first; fix non-sargable predicates; cover your JOIN columns with indexes |

---

> **Top 5 questions that always come up in senior backend interviews:**
> 1. Clustered vs non-clustered index (Q1)
> 2. Explain ACID with an example (Q12)
> 3. Isolation levels and when to use each (Q14)
> 4. How does EXPLAIN work and what does `type: ALL` mean? (Q19)
> 5. What is the N+1 problem? (Q20)
