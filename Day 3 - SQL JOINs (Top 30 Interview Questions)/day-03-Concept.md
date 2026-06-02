# Day 3 — SQL JOINs (Most Important for Interviews)

---

## 🚀 Focus:

### Master SQL JOINs — the most frequently asked topic in backend & data interviews.

---

## Why JOINs Matter

- Real-world databases are relational
- Most queries involve multiple tables
- Core skill for:
- Backend Developers
- Data Engineers
- Full Stack Developers

---

## 🧠 Types of JOINs

| JOIN Type | Description |
| --- | --- |
| INNER JOIN | Only matching records from both tables |
| LEFT JOIN | All rows from left table + matching rows from right |
| RIGHT JOIN | All rows from right table + matching rows from left |
| FULL JOIN | All rows from both tables (emulated via UNION in MySQL) |
| SELF JOIN | A table joined with itself |
| CROSS JOIN | Every row of left paired with every row of right (Cartesian product) |

---

## 📊 Visual Diagrams + Detailed Examples

> All examples use the `employees` and `departments` tables defined in the Sample Dataset section below.
>
> Quick reminder of the data:
> - **employees**: Alice(dept 1), Bob(dept 2), Charlie(dept NULL), David(dept 1), Eva(dept 3)
> - **departments**: 1=Engineering, 2=HR, 3=Finance

---

### 1️⃣ INNER JOIN

Returns **only rows that have a match in both tables**. Rows with no match on either side are dropped.

```
  employees          departments
  ┌─────────┐        ┌─────────────┐
  │ Alice   │────────│ Engineering │
  │ Bob     │────────│ HR          │
  │ Charlie │  ✗     │ Finance     │────── Eva
  │ David   │────────│             │
  │ Eva     │────────│             │
  └─────────┘        └─────────────┘
        ↑ Charlie has no dept → excluded
```

```
        employees              departments
      ╔══════════╗           ╔══════════════╗
      ║          ║███████████║              ║
      ║ (no dept)║███ RESULT ║(no employees)║
      ║          ║███████████║              ║
      ╚══════════╝           ╚══════════════╝
                  ↑ Only the overlapping middle
```

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

**Result:**

| name  | department_name |
| ----- | --------------- |
| Alice | Engineering     |
| David | Engineering     |
| Bob   | HR              |
| Eva   | Finance         |

> Charlie is excluded — `department_id` is NULL, so no match exists.

---

### 2️⃣ LEFT JOIN (LEFT OUTER JOIN)

Returns **all rows from the left table**, and the matched rows from the right table.
Where there is no match, right-side columns are `NULL`.

```
  employees (LEFT)       departments (RIGHT)
  ┌─────────┐            ┌─────────────┐
  │ Alice   │────────────│ Engineering │
  │ Bob     │────────────│ HR          │
  │ Charlie │──── NULL   │             │  ← no match, right side = NULL
  │ David   │────────────│ Engineering │
  │ Eva     │────────────│ Finance     │
  └─────────┘            └─────────────┘
```

```
        employees              departments
      ╔══════════╗           ╔══════════════╗
      ║          ║███████████║              ║
      ║ ALL ROWS ║███████████║  (matched)   ║
      ║          ║███████████║              ║
      ╚══════════╝           ╚══════════════╝
        ↑ Entire left table always included
```

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id;
```

**Result:**

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Bob     | HR              |
| Charlie | NULL            |
| David   | Engineering     |
| Eva     | Finance         |

> Charlie appears with `NULL` because no department matches.

**Classic trick — find unmatched rows only (anti-join):**

```sql
-- Employees with NO department
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

| name    |
| ------- |
| Charlie |

---

### 3️⃣ RIGHT JOIN (RIGHT OUTER JOIN)

Returns **all rows from the right table**, and the matched rows from the left table.
Where there is no match, left-side columns are `NULL`.

```
  employees (LEFT)       departments (RIGHT)
  ┌─────────┐            ┌─────────────┐
  │ Alice   │────────────│ Engineering │
  │ Bob     │────────────│ HR          │
  │ Eva     │────────────│ Finance     │
  │         │     NULL ──│ Marketing   │  ← new dept, no employees yet
  └─────────┘            └─────────────┘
```

```
        employees              departments
      ╔══════════╗           ╔══════════════╗
      ║          ║███████████║              ║
      ║ (matched)║███████████║   ALL ROWS   ║
      ║          ║███████████║              ║
      ╚══════════╝           ╚══════════════╝
                               ↑ Entire right table always included
```

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.id;
```

**Result** (using the existing dataset — every dept has at least one employee):

| name  | department_name |
| ----- | --------------- |
| Alice | Engineering     |
| David | Engineering     |
| Bob   | HR              |
| Eva   | Finance         |

> If a department existed with no employees, `name` would be `NULL` for that row.

**Anti-join variant — departments with NO employees:**

```sql
SELECT d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL;
```

---

### 4️⃣ FULL JOIN (FULL OUTER JOIN — MySQL workaround)

Returns **all rows from both tables**. Where there is no match on either side, the missing columns are `NULL`.

> MySQL does NOT support `FULL OUTER JOIN` directly. Emulate it with `LEFT JOIN UNION RIGHT JOIN`.

```
  employees (LEFT)       departments (RIGHT)
  ┌─────────┐            ┌─────────────┐
  │ Alice   │────────────│ Engineering │
  │ Bob     │────────────│ HR          │
  │ Charlie │──── NULL   │             │  ← Charlie: right side NULL
  │ David   │────────────│ Engineering │
  │ Eva     │────────────│ Finance     │
  │         │     NULL ──│ Marketing   │  ← Marketing: left side NULL
  └─────────┘            └─────────────┘
```

```
        employees              departments
      ╔══════════╗           ╔══════════════╗
      ║          ║███████████║              ║
      ║ ALL ROWS ║███████████║   ALL ROWS   ║
      ║          ║███████████║              ║
      ╚══════════╝           ╚══════════════╝
        ↑ Both sides fully included, NULLs fill gaps
```

```sql
-- LEFT side: all employees + their dept (NULL if no dept)
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

-- RIGHT side: all departments + their employees (NULL if no employees)
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Result:**

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Bob     | HR              |
| Charlie | NULL            |
| David   | Engineering     |
| Eva     | Finance         |

> `UNION` deduplicates. Use `UNION ALL` only if you intentionally want duplicates.

---

### 5️⃣ SELF JOIN

The table is **joined with itself** using aliases. Used to compare rows within the same table.

**Classic use case:** Employee → Manager hierarchy (both stored in the same `employees` table via `manager_id`).

```
  employees table used TWICE (as e and m)

  e (employee side)       m (manager side)
  ┌─────────────────┐     ┌─────────────────┐
  │ id=1  Alice     │─────│ (no manager)    │  manager_id=NULL
  │ id=2  Bob       │─────│ id=1 Alice      │  Bob's manager is Alice
  │ id=3  Charlie   │─────│ id=1 Alice      │  Charlie's manager is Alice
  │ id=4  David     │─────│ id=1 Alice      │  David's manager is Alice
  │ id=5  Eva       │─────│ id=2 Bob        │  Eva's manager is Bob
  └─────────────────┘     └─────────────────┘
```

```sql
SELECT e.name  AS employee,
       m.name  AS manager
FROM employees e
LEFT JOIN employees m         -- same table, aliased as m
ON e.manager_id = m.id;
```

**Result:**

| employee | manager |
| -------- | ------- |
| Alice    | NULL    |
| Bob      | Alice   |
| Charlie  | Alice   |
| David    | Alice   |
| Eva      | Bob     |

> Alice has `NULL` manager — she is the top-level employee.

**Finding peers (employees in the same department):**

```sql
SELECT e1.name AS emp1,
       e2.name AS emp2,
       e1.department_id
FROM employees e1
JOIN employees e2
ON  e1.department_id = e2.department_id
AND e1.id <> e2.id;           -- exclude pairing with self
```

---

### 6️⃣ CROSS JOIN

Returns the **Cartesian product** — every row from the left table paired with every row from the right table. No `ON` condition needed.

```
  employees (5 rows)   ×   departments (3 rows)   =   15 result rows

  Alice   × Engineering
  Alice   × HR
  Alice   × Finance
  Bob     × Engineering
  Bob     × HR
  Bob     × Finance
  ... and so on for all 5 employees
```

```
        employees              departments
      ╔══════════╗           ╔══════════════╗
      ║  5 rows  ║  × (all)  ║   3 rows     ║
      ║          ║───────────║              ║
      ╚══════════╝           ╚══════════════╝
                  = 5 × 3 = 15 rows output
```

```sql
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

**Partial result (15 total rows):**

| name  | department_name |
| ----- | --------------- |
| Alice | Engineering     |
| Alice | HR              |
| Alice | Finance         |
| Bob   | Engineering     |
| Bob   | HR              |
| Bob   | Finance         |
| ...   | ...             |

> Use cases: generating test data, building combination/schedule matrices.
> Avoid on large tables — row count explodes as `left_rows × right_rows`.

---

## 🗺️ All JOINs at a Glance

```
                   LEFT        RIGHT
                  TABLE        TABLE
                 ┌──────┐    ┌──────┐
                 │      │    │      │
  INNER JOIN     │   ░░░████░░░     │   ← only overlap
                 │      │    │      │
                 └──────┘    └──────┘

                 ┌──────┐    ┌──────┐
                 │      │    │      │
  LEFT JOIN      │██████████░░░     │   ← full left + overlap
                 │      │    │      │
                 └──────┘    └──────┘

                 ┌──────┐    ┌──────┐
                 │      │    │      │
  RIGHT JOIN     │   ░░░██████████  │   ← overlap + full right
                 │      │    │      │
                 └──────┘    └──────┘

                 ┌──────┐    ┌──────┐
                 │      │    │      │
  FULL JOIN      │████████████████  │   ← everything from both
                 │      │    │      │
                 └──────┘    └──────┘

  SELF JOIN      Table joined with itself (uses aliases)

  CROSS JOIN     Every row × every row  (no ON condition)
```

---

---

## Sample Dataset

---

### Employees Table

| id | name    | department_id | salary | manager_id |
| ---: | --- | ---: | ---: | --- |
| 1 | Alice   | 1   | 70000 | NULL |
| 2 | Bob     | 2   | 60000 | 1 |
| 3 | Charlie | NULL| 50000 | 1 |
| 4 | David   | 1   | 80000 | 1 |
| 5 | Eva     | 3   | 90000 | 2 |

```
CREATE TABLE employees (
    id INT,
    name VARCHAR(50),
    department_id INT,
    salary INT,
    manager_id INT
);

INSERT INTO employees VALUES
(1, 'Alice', 1, 70000, NULL),
(2, 'Bob', 2, 60000, 1),
(3, 'Charlie', NULL, 50000, 1),
(4, 'David', 1, 80000, 1),
(5, 'Eva', 3, 90000, 2);

```

### Departments Table

| id | department_name |
| ---: | --- |
| 1 | Engineering |
| 2 | HR |
| 3 | Finance |

```
CREATE TABLE departments (
    id INT,
    department_name VARCHAR(50)
);

INSERT INTO departments VALUES
(1, 'Engineering'),
(2, 'HR'),
(3, 'Finance');

```

### Customers Table

| customer_id | customer_name |
| ---: | --- |
| 1 | John |
| 2 | Mary |
| 3 | Steve |

```
CREATE TABLE customers (
    customer_id INT,
    customer_name VARCHAR(50)
);

INSERT INTO customers VALUES
(1, 'John'),
(2, 'Mary'),
(3, 'Steve');

```

### Orders Table

| order_id | customer_id | amount |
| ---: | ---: | ---: |
| 101 | 1 | 200 |
| 102 | 2 | 500 |
| 103 | 1 | 300 |

```
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount INT
);

INSERT INTO orders VALUES
(101, 1, 200),
(102, 2, 500),
(103, 1, 300);
```
---
### NOTE: 
- We didn't add the foreign key or index here on our 4 tables, 
- The Foreign key relation is not required for the join query learning or practicing.
- For a production-ready query, it is required. 
---

## 🔥 Top 30 SQL JOIN Interview Questions

> **Dataset used throughout:** `employees`, `departments`, `customers`, `orders` — defined in the Sample Dataset section above.

---

### Q1 — INNER JOIN: Employee names with their departments

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** Get each employee's name alongside their department name. Exclude employees who have no department assigned.

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

**Result:**

| name  | department_name |
| ----- | --------------- |
| Alice | Engineering     |
| David | Engineering     |
| Bob   | HR              |
| Eva   | Finance         |

> Charlie is excluded — his `department_id` is `NULL`, so no row in `departments` matches.

---

### Q2 — LEFT JOIN: All employees, with or without a department

**JOIN Type:** `LEFT JOIN` | **Tables:** `employees ↔ departments`

**Question:** List all employees. If an employee has no department, still show them with `NULL` in the department column.

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id;
```

**Result:**

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Bob     | HR              |
| Charlie | NULL            |
| David   | Engineering     |
| Eva     | Finance         |

> Charlie appears with `NULL` — LEFT JOIN preserves every row from the left table regardless of match.

---

### Q3 — RIGHT JOIN: All departments, even without employees

**JOIN Type:** `RIGHT JOIN` | **Tables:** `employees ↔ departments`

**Question:** List all departments. If a department has no employees yet, still show it with `NULL` for the employee name.

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.id;
```

**Result:**

| name  | department_name |
| ----- | --------------- |
| Alice | Engineering     |
| David | Engineering     |
| Bob   | HR              |
| Eva   | Finance         |

> In this dataset every department has at least one employee, so no `NULL` rows appear. Add a new dept (e.g., Marketing) with no employees to see `NULL` on the left side.

---

### Q4 — FULL JOIN: All employees AND all departments (MySQL trick)

**JOIN Type:** `LEFT JOIN` + `UNION` + `RIGHT JOIN` | **Tables:** `employees ↔ departments`

**Question:** Show every employee and every department — even if one side has no matching row on the other side. (`FULL OUTER JOIN` is not supported in MySQL; emulate it with `UNION`.)

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Result:**

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Bob     | HR              |
| Charlie | NULL            |
| David   | Engineering     |
| Eva     | Finance         |

> `UNION` removes duplicates automatically. Use `UNION ALL` only if you intentionally want duplicate rows.

---

### Q5 — Anti-JOIN: Employees with NO department

**JOIN Type:** `LEFT JOIN` + `IS NULL` filter | **Tables:** `employees ↔ departments`

**Question:** Find employees who have not been assigned to any department.

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id
WHERE d.id IS NULL;
```

**Result:**

| name    |
| ------- |
| Charlie |

> **Pattern:** LEFT JOIN + `WHERE right_table.pk IS NULL` is the standard anti-join pattern for finding unmatched rows.

---

### Q6 — Anti-JOIN: Departments with NO employees

**JOIN Type:** `LEFT JOIN` + `IS NULL` filter | **Tables:** `departments ↔ employees`

**Question:** Find departments that currently have no employees assigned.

```sql
SELECT d.department_name
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
WHERE e.id IS NULL;
```

**Result:**

| department_name |
| --------------- |
| *(empty)*       |

> In this dataset every department has at least one employee. Insert a row like `(4, 'Marketing')` into `departments` to see it appear here.

---

### Q7 — JOIN with WHERE filter: High earners and their departments

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** List employees earning more than ₹70,000 along with their department names.

```sql
SELECT e.name, d.department_name, e.salary
FROM employees e
JOIN departments d
ON e.department_id = d.id
WHERE e.salary > 70000;
```

**Result:**

| name  | department_name | salary |
| ----- | --------------- | -----: |
| David | Engineering     |  80000 |
| Eva   | Finance         |  90000 |

> The `WHERE` clause filters rows **after** the JOIN is resolved. Alice (70000) is excluded because `70000 > 70000` is false.

---

### Q8 — JOIN + GROUP BY: Employee count per department

**JOIN Type:** `LEFT JOIN` | **Tables:** `departments ↔ employees`

**Question:** Show how many employees each department has. Include departments with zero employees.

```sql
SELECT d.department_name,
       COUNT(e.id) AS employee_count
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name;
```

**Result:**

| department_name | employee_count |
| --------------- | -------------: |
| Engineering     |              2 |
| HR              |              1 |
| Finance         |              1 |

> Use `LEFT JOIN` (not `INNER JOIN`) so departments with zero employees still appear with `count = 0`.

---

### Q9 — JOIN + MAX: Highest salary per department

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** Find the highest salary in each department.

```sql
SELECT d.department_name,
       MAX(e.salary) AS max_salary
FROM employees e
JOIN departments d
ON e.department_id = d.id
GROUP BY d.department_name;
```

**Result:**

| department_name | max_salary |
| --------------- | ---------: |
| Engineering     |      80000 |
| HR              |      60000 |
| Finance         |      90000 |

---

### Q10 — SELF JOIN: Employee → Manager hierarchy

**JOIN Type:** `SELF JOIN` (LEFT JOIN) | **Table:** `employees` joined with itself

**Question:** Show each employee along with their manager's name. Employees with no manager should still appear.

```sql
SELECT e.name  AS employee,
       m.name  AS manager
FROM employees e
LEFT JOIN employees m        -- same table aliased as "m" for manager side
ON e.manager_id = m.id;
```

**Result:**

| employee | manager |
| -------- | ------- |
| Alice    | NULL    |
| Bob      | Alice   |
| Charlie  | Alice   |
| David    | Alice   |
| Eva      | Bob     |

> Alice has no manager — she is the root of the hierarchy. LEFT JOIN ensures she still appears.

---

### Q11 — INNER JOIN: Customers who have placed orders

**JOIN Type:** `INNER JOIN` | **Tables:** `customers ↔ orders`

**Question:** List the names of customers who have placed at least one order (show each order as a separate row).

```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | amount |
| ------------- | -------: | -----: |
| John          |      101 |    200 |
| John          |      103 |    300 |
| Mary          |      102 |    500 |

> Steve has no orders and is excluded by the INNER JOIN. John appears twice — once per order.

---

### Q12 — Anti-JOIN: Customers with NO orders

**JOIN Type:** `LEFT JOIN` + `IS NULL` filter | **Tables:** `customers ↔ orders`

**Question:** Find customers who have never placed an order.

```sql
SELECT c.customer_name
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Result:**

| customer_name |
| ------------- |
| Steve         |

---

### Q13 — JOIN + SUM: Total order amount per customer

**JOIN Type:** `INNER JOIN` | **Tables:** `customers ↔ orders`

**Question:** Calculate the total amount each customer has spent across all their orders.

```sql
SELECT c.customer_name,
       SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```

**Result:**

| customer_name | total_spent |
| ------------- | ----------: |
| John          |         500 |
| Mary          |         500 |

> John placed two orders (200 + 300 = 500). Steve is excluded — no orders to sum.

---

### Q14 — JOIN + MAX: Largest single order per customer

**JOIN Type:** `INNER JOIN` | **Tables:** `customers ↔ orders`

**Question:** For each customer who has placed orders, find the value of their largest single order.

```sql
SELECT c.customer_name,
       MAX(o.amount) AS largest_order
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```

**Result:**

| customer_name | largest_order |
| ------------- | ------------: |
| John          |           300 |
| Mary          |           500 |

---

### Q15 — Three-table JOIN: Employee, department, and order data

**JOIN Type:** `INNER JOIN` (chained) | **Tables:** `employees ↔ departments ↔ orders`

**Question:** List employees, their department, and any orders associated with their employee ID. Only show employees who have a department and a matching order.

```sql
SELECT e.name, d.department_name, o.order_id, o.amount
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN orders o     ON e.id = o.customer_id;
```

**Result:**

| name  | department_name | order_id | amount |
| ----- | --------------- | -------: | -----: |
| Alice | Engineering     |      101 |    200 |
| Alice | Engineering     |      103 |    300 |
| Bob   | HR              |      102 |    500 |

> Each JOIN adds another table to the result set. Chain as many as needed; order of JOINs rarely affects the final result but can affect performance.

---

### Q16 — SELF JOIN: Find employees with identical salaries

**JOIN Type:** `SELF JOIN` (INNER JOIN) | **Table:** `employees` joined with itself

**Question:** Find pairs of employees who earn exactly the same salary.

```sql
SELECT a.name  AS emp1,
       b.name  AS emp2,
       a.salary
FROM employees a
JOIN employees b
ON  a.salary = b.salary
AND a.id <> b.id;           -- prevent an employee pairing with themselves
```

**Result:**

| emp1 | emp2 | salary |
| ---- | ---- | -----: |
| *(empty — all salaries are unique in this dataset)* | | |

> The `a.id <> b.id` condition is essential — without it every employee would match themselves.

---

### Q17 — JOIN with multiple ON conditions

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** List employees who belong to a department AND earn more than ₹60,000. Apply both conditions inside the `ON` clause.

```sql
SELECT e.name, d.department_name, e.salary
FROM employees e
JOIN departments d
ON  e.department_id = d.id
AND e.salary > 60000;
```

**Result:**

| name  | department_name | salary |
| ----- | --------------- | -----: |
| Alice | Engineering     |  70000 |
| David | Engineering     |  80000 |
| Eva   | Finance         |  90000 |

> Adding filters inside `ON` is equivalent to putting them in `WHERE` for an INNER JOIN, but behaves **differently** for OUTER JOINs — `ON` filters before joining; `WHERE` filters after.

---

### Q18 — CROSS JOIN: All employee–department combinations

**JOIN Type:** `CROSS JOIN` | **Tables:** `employees × departments`

**Question:** Generate every possible pairing of employee and department (5 employees × 3 departments = 15 rows).

```sql
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

**Partial Result (15 rows total):**

| name    | department_name |
| ------- | --------------- |
| Alice   | Engineering     |
| Alice   | HR              |
| Alice   | Finance         |
| Bob     | Engineering     |
| Bob     | HR              |
| Bob     | Finance         |
| Charlie | Engineering     |
| ...     | ...             |

> Common use case: generating a schedule matrix or test-data combinations. Dangerous on large tables — row count is `left × right`.

---

### Q19 — USING clause: Simplified JOIN syntax

**JOIN Type:** `INNER JOIN` with `USING` | **Tables:** `customers ↔ orders`

**Question:** Join customers with their orders using the `USING` shorthand (works when both tables share an identical column name).

```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
JOIN orders o
USING (customer_id);         -- both tables have a column named customer_id
```

**Result:**

| customer_name | order_id | amount |
| ------------- | -------: | -----: |
| John          |      101 |    200 |
| John          |      103 |    300 |
| Mary          |      102 |    500 |

> `USING(col)` is a shortcut for `ON table1.col = table2.col`. It only works when **both tables have the exact same column name**. That is why `employees ↔ departments` cannot use `USING` here — employees has `department_id` but departments has `id`.

---

### Q20 — JOIN + WHERE: Employees in a specific department

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** List all employees who work in the Engineering department.

```sql
SELECT e.name, e.salary
FROM employees e
JOIN departments d
ON e.department_id = d.id
WHERE d.department_name = 'Engineering';
```

**Result:**

| name  | salary |
| ----- | -----: |
| Alice |  70000 |
| David |  80000 |

---


### Q21 — LEFT JOIN + multiple aggregates: Department summary

**JOIN Type:** `LEFT JOIN` | **Tables:** `departments ↔ employees`

**Question:** For each department, show the number of employees, average salary, and highest salary. Include departments with no employees.

```sql
SELECT d.department_name,
       COUNT(e.id)       AS headcount,
       AVG(e.salary)     AS avg_salary,
       MAX(e.salary)     AS max_salary
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name;
```

**Result:**

| department_name | headcount | avg_salary | max_salary |
| --------------- | --------: | ---------: | ---------: |
| Engineering     |         2 |    75000.0 |      80000 |
| HR              |         1 |    60000.0 |      60000 |
| Finance         |         1 |    90000.0 |      90000 |

---

### Q22 — SELF JOIN: Pairs of employees in the same department

**JOIN Type:** `SELF JOIN` (INNER JOIN) | **Table:** `employees` joined with itself

**Question:** Find all distinct pairs of employees who belong to the same department.

```sql
SELECT e1.name          AS employee_1,
       e2.name          AS employee_2,
       e1.department_id
FROM employees e1
JOIN employees e2
ON  e1.department_id = e2.department_id
AND e1.id < e2.id;      -- use < instead of <> to avoid (A,B) and (B,A) duplicates
```

**Result:**

| employee_1 | employee_2 | department_id |
| ---------- | ---------- | ------------: |
| Alice      | David      |             1 |

> Using `e1.id < e2.id` instead of `e1.id <> e2.id` ensures each pair appears only once.

---

### Q23 — JOIN + SUM + LIMIT: Department with the highest total salary bill

**JOIN Type:** `INNER JOIN` | **Tables:** `employees ↔ departments`

**Question:** Which single department has the largest combined salary expenditure?

```sql
SELECT d.department_name,
       SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d
ON e.department_id = d.id
GROUP BY d.department_name
ORDER BY total_salary DESC
LIMIT 1;
```

**Result:**

| department_name | total_salary |
| --------------- | -----------: |
| Engineering     |       150000 |

> Engineering: 70000 (Alice) + 80000 (David) = 150000. Finance is second at 90000.

---


### Q24 — Anti-JOIN across unrelated tables: Employees with no matching orders

**JOIN Type:** `LEFT JOIN` + `IS NULL` filter | **Tables:** `employees ↔ orders`

**Question:** Find employees whose `id` does not appear as a `customer_id` in the orders table.

```sql
SELECT e.name
FROM employees e
LEFT JOIN orders o
ON e.id = o.customer_id
WHERE o.order_id IS NULL;
```

**Result:**

| name    |
| ------- |
| Charlie |
| David   |
| Eva     |

> Orders contain `customer_id` values 1 (Alice) and 2 (Bob), so Charlie (id=3), David (id=4), and Eva (id=5) have no match.

---

### Q25 — JOIN + HAVING: Departments with more than N employees

**JOIN Type:** `INNER JOIN` | **Tables:** `departments ↔ employees`

**Question:** List departments that have more than 2 employees.

```sql
SELECT d.department_name,
       COUNT(e.id) AS employee_count
FROM departments d
JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name
HAVING COUNT(e.id) > 2;
```

**Result:**

| department_name | employee_count |
| --------------- | -------------: |
| *(empty)*       |                |

> No department in this dataset has more than 2 employees — Engineering has exactly 2. Change `> 2` to `>= 2` or add more employees to see results. The key pattern: **`WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.**

---

# 🎯 Most Important Interview Patterns — Theory Q&A

> These are the conceptual questions interviewers ask **before or after** showing you a query. Nail these and you stand out.

---

## 1. INNER JOIN vs LEFT JOIN

---

**Q:1 What is the difference between INNER JOIN and LEFT JOIN?**

**A:**

| | INNER JOIN | LEFT JOIN |
|---|---|---|
| Rows returned | Only rows with a match in **both** tables | **All** rows from the left table; `NULL` for unmatched right-side columns |
| Unmatched left rows | Dropped | Kept (right columns = `NULL`) |
| Unmatched right rows | Dropped | Dropped |
| Use when | You only care about related data | You want to keep all left-table records regardless of match |

```sql
-- INNER JOIN: only matched employees (Charlie excluded)
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN: all employees (Charlie shown with NULL dept)
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

---

**Q:2 When would you choose LEFT JOIN over INNER JOIN in production?**

**A:** Use LEFT JOIN when:
- You need a **complete list** of one entity (e.g., all customers) regardless of whether related data exists (e.g., orders).
- You are building **reports** where missing data must be visible as `NULL` or `0`, not simply omitted.
- You want to detect **gaps** — rows with no matching counterpart (anti-join pattern).

---

**Q:3 Does the order of tables matter in INNER JOIN?**

**A:** No — `A INNER JOIN B` and `B INNER JOIN A` produce the same rows (the result set is symmetric). Column order in `SELECT` may differ, but the row set is identical.

For `LEFT JOIN` and `RIGHT JOIN`, order **does** matter — the table on the left is the "preserved" side.

---

## 2. LEFT JOIN + NULL Filtering (Anti-Join)

---

**Q:4 What is an anti-join and how do you write one in MySQL?**

**A:** An anti-join returns rows from one table that have **no matching row** in another table. MySQL has no `ANTI JOIN` keyword, so you emulate it with LEFT JOIN + `IS NULL`:

```sql
-- Employees with NO department
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;          -- d.id is NULL only when no match was found
```

The key insight: after a LEFT JOIN, any column from the **right table's primary key** will be `NULL` only for rows that had no match.

---

**Q:5 Why must you filter on a NOT NULL column (like a primary key) and not just any column?**

**A:** If you filter on a nullable column, you can't tell whether `NULL` means "no match" or "the column genuinely stores NULL in the database". Always filter on a column that is defined `NOT NULL` in the right table — a primary key is the safest choice.

```sql
-- WRONG (department_name could legitimately be NULL)
WHERE d.department_name IS NULL

-- CORRECT (primary key is always NOT NULL — NULL means no match)
WHERE d.id IS NULL
```

---

**Q:6 What is the difference between NOT IN and LEFT JOIN + IS NULL for finding missing records?**

**A:**

| Approach | Behaviour with NULLs | Performance |
|---|---|---|
| `NOT IN (subquery)` | Returns **no rows** if the subquery contains even one `NULL` | Can be slow on large sets |
| `LEFT JOIN + IS NULL` | Safe — NULLs don't affect correctness | Usually faster; optimizer can use indexes |
| `NOT EXISTS` | Safe and often fastest | Best for correlated checks |

```sql
-- DANGEROUS if subquery can return NULL
SELECT name FROM employees
WHERE department_id NOT IN (SELECT id FROM departments);

-- SAFE alternative
SELECT e.name FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

---

## 3. SELF JOIN

---

**Q:7 What is a SELF JOIN and when do you use it?**

**A:** A SELF JOIN joins a table to itself using two different aliases. The table is physically the same, but the aliases let the query treat it as two separate virtual tables.

**Use cases:**
- **Hierarchy / tree structures** — employee → manager (both in the same table)
- **Peer comparison** — finding rows that share a common value (same department, same salary)
- **Sequence gaps** — finding consecutive records

```sql
-- Employee-manager hierarchy
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

**Q:8 What happens if you forget to add the `id <> id` or `id < id` condition in a SELF JOIN?**

**A:**
- Without any guard: every employee matches themselves, producing duplicate/redundant rows (Alice paired with Alice, Bob paired with Bob, etc.).
- Use `e1.id <> e2.id` to exclude self-pairing (but still gives (Alice, David) AND (David, Alice) for the same pair).
- Use `e1.id < e2.id` to get each pair only once — the standard approach for finding peer combinations.

---

## 4. Aggregation with JOIN

---

**Q:9 Why must you use LEFT JOIN instead of INNER JOIN when counting rows per group (including groups with zero)?**

**A:** INNER JOIN drops rows with no match, so groups with zero related records disappear from the result. LEFT JOIN preserves them, and `COUNT(right_table.pk)` returns `0` for those groups because `COUNT` ignores `NULL` values.

```sql
-- WRONG: departments with 0 employees won't appear
SELECT d.department_name, COUNT(e.id)
FROM departments d
INNER JOIN employees e ON d.id = e.department_id
GROUP BY d.department_name;

-- CORRECT: all departments appear, count=0 for empty ones
SELECT d.department_name, COUNT(e.id)
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.department_name;
```

---

**Q:10 What is the difference between WHERE and HAVING?**

**A:**

| | WHERE | HAVING |
|---|---|---|
| Runs | Before `GROUP BY` (filters individual rows) | After `GROUP BY` (filters entire groups) |
| Can use aggregate functions | No | Yes |
| Use for | Row-level conditions | Group-level conditions |

```sql
-- WHERE: filter rows first, then group
SELECT department_id, AVG(salary)
FROM employees
WHERE salary > 50000           -- removes individual rows before grouping
GROUP BY department_id;

-- HAVING: group first, then filter groups
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 70000;   -- filters entire groups after aggregation
```

---

**Q:11 Can you use both WHERE and HAVING in the same query?**

**A:** Yes — and it is common. `WHERE` filters rows before grouping; `HAVING` filters the resulting groups.

```sql
SELECT d.department_name, COUNT(e.id) AS headcount
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
WHERE e.salary > 60000           -- only consider employees earning > 60k
GROUP BY d.department_name
HAVING COUNT(e.id) >= 1;         -- only show departments with at least 1 such employee
```

---

## 5. Multi-Table JOIN

---

**Q:12 How do you join more than two tables?**

**A:** Chain JOIN clauses — each one adds another table to the result set. The optimizer decides the physical join order.

```sql
SELECT e.name, d.department_name, o.order_id, o.amount
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN orders o      ON e.id = o.customer_id;
```

Each `JOIN` line references one new table. You can mix `INNER JOIN`, `LEFT JOIN`, etc. in the same query.

---

**Q:13 Does the order of JOINs in a multi-table query affect the result?**

**A:** For INNER JOINs — **no**, the result is mathematically the same regardless of join order. The MySQL optimizer may reorder them internally for performance.

For OUTER JOINs — **yes**, order can matter because which table is the "preserved" side changes the rows that survive with `NULL`s. Always think carefully about which table you want all rows from.

---

**Q:14 What is a JOIN condition vs a filter condition, and why does it matter for OUTER JOINs?**

**A:**
- **JOIN condition (`ON`)** — determines which rows are matched between tables. Evaluated before the outer join produces `NULL`-padded rows.
- **Filter condition (`WHERE`)** — applied after the join, discards rows from the already-joined result set.

For OUTER JOINs this distinction is critical:

```sql
-- This keeps all departments (right table) even without matching employees:
SELECT d.department_name, e.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
    AND e.salary > 70000;        -- ON: filtered before join — unmatched depts still appear

-- This turns the RIGHT JOIN into an INNER JOIN effectively:
SELECT d.department_name, e.name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.salary > 70000;          -- WHERE: removes NULL rows → kills outer join behaviour
```

---

## 6. Finding Missing Records

---

**Q:15 What are the three ways to find rows in table A that have no match in table B?**

**A:**

```sql
-- Method 1: LEFT JOIN + IS NULL (most common, good performance)
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;

-- Method 2: NOT EXISTS (correlated subquery — often fastest)
SELECT e.name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM departments d
    WHERE d.id = e.department_id
);

-- Method 3: NOT IN (avoid if subquery can return NULL)
SELECT e.name
FROM employees e
WHERE department_id NOT IN (
    SELECT id FROM departments
);
```

**Rule of thumb:** Prefer `NOT EXISTS` or `LEFT JOIN + IS NULL`. Avoid `NOT IN` unless you are certain the subquery can never return `NULL`.

---

**Q:16 Why does `NOT IN` return no rows when the subquery contains a NULL?**

**A:** SQL uses three-valued logic (TRUE / FALSE / UNKNOWN). Any comparison with `NULL` yields `UNKNOWN`, and `WHERE UNKNOWN` is treated as false — so the row is excluded.

```sql
-- If departments had id = NULL somewhere:
WHERE department_id NOT IN (1, 2, NULL)
-- is evaluated as:
WHERE department_id <> 1 AND department_id <> 2 AND department_id <> NULL
-- the last condition is UNKNOWN → entire expression is UNKNOWN → row excluded
```

This is why `NOT IN` silently returns zero rows when the subquery has any `NULL` value — a notoriously subtle bug.

---

## 7. Correlated Subqueries

---

**Q:17 What is a correlated subquery?**

**A:** A subquery that **references a column from the outer query**. It is re-executed once for every row the outer query processes, making it logically equivalent to a nested loop.

```sql
-- For each employee (outer), run the subquery for that employee's department
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id   -- ← correlates to outer "e"
);
```

---

**Q:18 What is the performance concern with correlated subqueries, and how can you rewrite them?**

**A:** Since the subquery runs once per outer row, it is O(N × M) in the worst case. For large tables this is slow. Rewrite using a JOIN + derived table or CTE:

```sql
-- Correlated subquery: runs AVG once per employee row
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE department_id = e.department_id
);

-- Better: compute all dept averages once, then JOIN
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_sal;
```

---

**Q:19 What is the difference between EXISTS and IN?**

**A:**

| | `IN` | `EXISTS` |
|---|---|---|
| Evaluates | Entire subquery result set | Stops as soon as one match is found (short-circuits) |
| NULL handling | Dangerous — `NULL` in list → unexpected results | Safe — returns TRUE/FALSE, not affected by NULL values |
| Best for | Small, non-nullable result sets | Large tables; correlated checks |

```sql
-- IN: get the whole set first, then check membership
SELECT name FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE department_name = 'Engineering');

-- EXISTS: stops at first match — more efficient on large tables
SELECT name FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.id = e.department_id AND d.department_name = 'Engineering'
);
```

---

## 8. Duplicate Detection

---

**Q:20 How do you find duplicate rows in a table?**

**A:** Use `GROUP BY` on the columns that define a "duplicate", then `HAVING COUNT(*) > 1`:

```sql
-- Find duplicate salaries
SELECT salary, COUNT(*) AS occurrences
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;

-- Find employees with the exact same name AND department
SELECT name, department_id, COUNT(*) AS occurrences
FROM employees
GROUP BY name, department_id
HAVING COUNT(*) > 1;
```

---

**Q:21 How do you find duplicate rows using a SELF JOIN?**

**A:** Join the table with itself on the duplicate-defining columns, then exclude self-pairing:

```sql
SELECT a.id AS id1, b.id AS id2, a.salary
FROM employees a
JOIN employees b
ON  a.salary      = b.salary
AND a.department_id = b.department_id
AND a.id < b.id;           -- < gives each pair once only
```

This gives you the actual IDs of duplicate pairs, which is useful when you need to delete or merge them.

---

**Q:22 How do you delete duplicate rows but keep one copy?**

**A:** Use `DELETE` with a self-join or a subquery targeting the higher (or lower) `id`:

```sql
-- Keep the row with the smallest id; delete all higher-id duplicates
DELETE e1
FROM employees e1
JOIN employees e2
ON  e1.salary = e2.salary
AND e1.id > e2.id;         -- delete e1 when a "better" copy (lower id) exists
```

---

## 9. Additional High-Value Theory Questions

---

**Q:23 What is the difference between `UNION` and `UNION ALL`?**

**A:**
- `UNION` — combines result sets and **removes duplicate rows** (runs a distinct operation, which is slower).
- `UNION ALL` — combines result sets and **keeps all rows including duplicates** (no deduplication, faster).

Use `UNION ALL` when you know rows are already distinct or when duplicates are intentional. Use `UNION` only when you need deduplication.

---

**Q:24 What is a JOIN vs a subquery — when do you use each?**

**A:**

| Situation | Prefer |
|---|---|
| Need columns from both tables in `SELECT` | JOIN |
| Just checking existence of a related row | `EXISTS` subquery |
| Aggregating related data before joining | CTE or derived table, then JOIN |
| One-time lookup into a small table | `IN` subquery |
| Complex filtering on grouped data | JOIN + `HAVING` |

As a rule: JOINs are generally more readable and the optimizer handles them well. Subqueries are useful for encapsulating a calculation that feeds into a comparison.

---

**Q:25 What is a derived table (inline view)?**

**A:** A subquery in the `FROM` clause — treated as a temporary table for the duration of the query. Useful for pre-aggregating data before joining.

```sql
SELECT e.name, dept_stats.avg_sal
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
) AS dept_stats                          -- derived table alias is required
ON e.department_id = dept_stats.department_id;
```

---

**Q:26 What is a CTE (`WITH` clause) and how is it different from a derived table?**

**A:** A CTE (Common Table Expression) defines a named temporary result set at the top of the query. It is equivalent to a derived table but:
- Can be referenced **multiple times** in the same query.
- Is defined **once** at the top, making complex queries easier to read.
- In MySQL 8.0, CTEs can be **recursive** (useful for tree/hierarchy traversal).

```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
SELECT e.name, e.salary, da.avg_sal
FROM employees e
JOIN dept_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_sal;
```

---

**Q:27 What is a Cartesian product and when does it accidentally happen?**

**A:** A Cartesian product pairs every row from one table with every row from another — producing `N × M` rows. It happens intentionally with `CROSS JOIN`, but accidentally when you:
- Forget the `ON` condition in a JOIN.
- Write an old-style comma join (`FROM tableA, tableB`) without a `WHERE` to link them.

```sql
-- Accidental cartesian product (old style — avoid):
SELECT * FROM employees, departments;  -- 5 × 3 = 15 rows, no filter

-- Correct:
SELECT * FROM employees e
JOIN departments d ON e.department_id = d.id;
```

---

**Q:28 What is the execution order of a SQL SELECT statement?**

**A:** SQL is declarative — you write `SELECT` first, but the engine evaluates clauses in this logical order:

```
1. FROM          — identify tables
2. JOIN / ON     — combine tables
3. WHERE         — filter individual rows
4. GROUP BY      — group rows
5. HAVING        — filter groups
6. SELECT        — compute output columns
7. DISTINCT      — remove duplicates (if used)
8. ORDER BY      — sort
9. LIMIT/OFFSET  — paginate
```

This order explains **why** you cannot use a `SELECT` alias in a `WHERE` clause (the alias doesn't exist yet when `WHERE` runs) but **can** use it in `ORDER BY` (which runs after `SELECT`).

---

## 🧪 Practice Tips

- Write every query from scratch without looking at the solution — recall is more valuable than recognition.
- After writing a query, explain **out loud** which rows are dropped at each JOIN step.
- For every LEFT JOIN you write, ask: "Do I need an anti-join filter (`IS NULL`) here, or do I want all rows?"
- Practice explaining the difference between `ON` and `WHERE` for outer joins — this is a common follow-up question.

---
