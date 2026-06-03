# 📘 Day 4 — Advanced SQL: Subqueries & Window Functions

> **Focus:** The two most-tested advanced SQL topics in product-based company interviews.

---

## 📋 What You Will Learn

| # | Topic | Concept |
|---|-------|---------|
| Part 1 | Subqueries | Scalar · Column · Correlated · EXISTS · NOT EXISTS |
| Part 2 | Window Functions | ROW_NUMBER · RANK · DENSE_RANK · LAG · LEAD · SUM OVER |
| Part 3 | Theory Q&A | 10 interview questions with detailed answers |
| Part 4 | Real Interview Problems | Second highest salary · Top-N per group · Duplicates |

---

## 📑 Index of Practice Questions

| Q# | Topic | Question |
|----|-------|----------|
| Q1 | Subquery — Scalar | Employees earning above overall average salary |
| Q2 | Subquery — Scalar | Employee with the highest salary |
| Q3 | Subquery — Single Value | Employees in the same department as John |
| Q4 | Subquery — IN | All employees in departments with more than one member |
| Q5 | Correlated Subquery | Employees earning above their own department average |
| Q6 | EXISTS | Employees who have at least one colleague in their department |
| Q7 | NOT EXISTS | Employees who are the sole member of their department |
| Q8 | Subquery in SELECT | Show each employee alongside their department's average salary |
| Q9 | ROW_NUMBER() | Rank employees within each department by salary |
| Q10 | ROW_NUMBER() — Top N | Top 2 highest-paid employees per department |
| Q11 | RANK() | Global salary rank (gaps on ties) |
| Q12 | DENSE_RANK() | Global salary rank (no gaps on ties) |
| Q13 | RANK vs DENSE_RANK | Side-by-side comparison |
| Q14 | DENSE_RANK — Nth salary | Second highest distinct salary |
| Q15 | DENSE_RANK — Nth salary | Nth highest distinct salary (parameterised) |
| Q16 | LAG() | Each employee's salary vs the previous employee |
| Q17 | LEAD() | Each employee's salary vs the next employee |
| Q18 | SUM OVER — Running Total | Global running salary total |
| Q19 | SUM OVER — Running Total | Running total within each department |
| Q20 | SUM OVER — Partition | Department salary total repeated on every row |

---

## 🧱 Sample Dataset

> **Note:** Lisa (id = 6) is added to the IT department with the same salary as Sarah. This creates a tie that makes **RANK vs DENSE_RANK** examples meaningful.

```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(50),
    department VARCHAR(50),
    salary     INT
);

INSERT INTO employees VALUES
(1, 'John',  'IT',    70000),
(2, 'Sarah', 'IT',    90000),
(3, 'Mike',  'HR',    60000),
(4, 'Anna',  'HR',    75000),
(5, 'David', 'Sales', 80000),
(6, 'Lisa',  'IT',    90000);   -- same salary as Sarah → creates a tie
```

**Data at a glance:**

| id | name  | department | salary |
| --: | ----- | ---------- | -----: |
| 1  | John  | IT         | 70,000 |
| 2  | Sarah | IT         | 90,000 |
| 3  | Mike  | HR         | 60,000 |
| 4  | Anna  | HR         | 75,000 |
| 5  | David | Sales      | 80,000 |
| 6  | Lisa  | IT         | 90,000 |

**Quick stats:**
- Overall average salary = (70k + 90k + 60k + 75k + 80k + 90k) / 6 = **77,500**
- IT average = (70k + 90k + 90k) / 3 = **83,333**
- HR average = (60k + 75k) / 2 = **67,500**
- Sales average = **80,000**

---

---

# 🔹 Part 1 — Subqueries

---

## 📖 Theory: What is a Subquery?

A **subquery** (also called an inner query or nested query) is a `SELECT` statement written **inside another SQL statement**. The outer query uses the result of the inner query.

### Types of Subqueries

| Type | Returns | Used In | Example Use |
|------|---------|---------|-------------|
| **Scalar** | Single value (1 row × 1 col) | `WHERE`, `SELECT`, `HAVING` | `WHERE salary > (SELECT AVG(...))` |
| **Column** | Single column, multiple rows | `IN`, `NOT IN`, `ANY`, `ALL` | `WHERE dept IN (SELECT dept ...)` |
| **Row** | Single row, multiple columns | `WHERE (a, b) = (SELECT ...)` | Row comparison |
| **Correlated** | Depends on outer row | `WHERE EXISTS`, comparison | Per-row filtering |

### Where Can a Subquery Appear?

```
SELECT  ← scalar subquery allowed here
FROM    ← derived table (inline view)
WHERE   ← most common placement
HAVING  ← filter groups based on aggregate subquery
```

### Syntax Templates

```sql
-- Scalar subquery in WHERE
SELECT ... FROM t WHERE col > (SELECT AGG(col) FROM t);

-- Column subquery with IN
SELECT ... FROM t WHERE col IN (SELECT col FROM other_t WHERE ...);

-- Derived table in FROM
SELECT * FROM (SELECT ...) AS alias;

-- Correlated subquery
SELECT ... FROM t outer WHERE val > (SELECT AGG(val) FROM t WHERE t.key = outer.key);
```

### Key Rules

- A scalar subquery **must return exactly one row and one column** — if it returns more, MySQL throws an error.
- Subqueries in `IN` can return multiple rows but must be **one column wide**.
- Always alias a derived table in `FROM`: `(SELECT ...) AS alias` — alias is mandatory in MySQL.
- Use `EXISTS` when you only need to know **whether a match exists**, not what the values are.

---

## Q1 — Scalar Subquery: Employees earning above average salary

**Question:** List all employees whose salary is higher than the company-wide average salary.

```sql
SELECT name, department, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)           -- returns 77500 (scalar)
    FROM employees
);
```

**Result:** *(AVG = 77,500)*

| name  | department | salary |
| ----- | ---------- | -----: |
| Sarah | IT         | 90,000 |
| David | Sales      | 80,000 |
| Lisa  | IT         | 90,000 |

> The inner query computes one value (77,500). The outer query uses it as a filter. Anna (75,000) and John (70,000) are excluded.

---

## Q2 — Scalar Subquery: Employee(s) with the highest salary

**Question:** Find all employees who earn the maximum salary in the company.

```sql
SELECT id, name, department, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)           -- returns 90000
    FROM employees
);
```

**Result:**

| id | name  | department | salary |
| --: | ----- | ---------- | -----: |
| 2  | Sarah | IT         | 90,000 |
| 6  | Lisa  | IT         | 90,000 |

> Using `= MAX(...)` (not `ORDER BY LIMIT 1`) correctly handles ties — both Sarah and Lisa are returned.

---

## Q3 — Single-Value Subquery: Employees in the same department as John

**Question:** List all employees (including John) who belong to the same department as John.

```sql
SELECT id, name, department, salary
FROM employees
WHERE department = (
    SELECT department            -- returns 'IT' (single value)
    FROM employees
    WHERE name = 'John'
);
```

**Result:**

| id | name  | department | salary |
| --: | ----- | ---------- | -----: |
| 1  | John  | IT         | 70,000 |
| 2  | Sarah | IT         | 90,000 |
| 6  | Lisa  | IT         | 90,000 |

> If `name = 'John'` returned more than one row, this query would fail with "Subquery returns more than 1 row". Use `IN` instead of `=` when the subquery might return multiple values.

---

## Q4 — Column Subquery (IN): Employees in departments with more than one member

**Question:** List all employees who belong to a department that has at least 2 employees.

```sql
SELECT id, name, department, salary
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 1          -- IT(3), HR(2) qualify; Sales(1) does not
);
```

**Result:**

| id | name  | department | salary |
| --: | ----- | ---------- | -----: |
| 1  | John  | IT         | 70,000 |
| 2  | Sarah | IT         | 90,000 |
| 3  | Mike  | HR         | 60,000 |
| 4  | Anna  | HR         | 75,000 |
| 6  | Lisa  | IT         | 90,000 |

> David (Sales) is excluded — Sales has only 1 employee. The inner query returns `('IT', 'HR')` and the outer query matches against that list.

---

## Q5 — Correlated Subquery: Employees earning above their department average

**Question:** Find employees whose salary is higher than the average salary of their own department.

```sql
SELECT name, department, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e1.department   -- correlation: links to outer row
);
```

**Result:** *(IT avg = 83,333 | HR avg = 67,500 | Sales avg = 80,000)*

| name  | department | salary |
| ----- | ---------- | -----: |
| Sarah | IT         | 90,000 |
| Anna  | HR         | 75,000 |
| Lisa  | IT         | 90,000 |

> The inner query runs **once per outer row**, each time filtering to the current employee's department. John (70k < 83k), Mike (60k < 67.5k), and David (80k = 80k, not greater) are excluded.

---

## Q6 — EXISTS: Employees who have at least one colleague in their department

**Question:** Find all employees who are **not** the only person in their department (i.e., at least one other person shares their department).

```sql
SELECT name, department
FROM employees e1
WHERE EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.department = e1.department   -- same department
      AND e2.id <> e1.id                  -- but not the same person
);
```

**Result:**

| name  | department |
| ----- | ---------- |
| John  | IT         |
| Sarah | IT         |
| Mike  | HR         |
| Anna  | HR         |
| Lisa  | IT         |

> David (Sales) is excluded — no other employee shares his department. `EXISTS` stops as soon as one match is found — it is faster than `IN` on large tables because it short-circuits.

---

## Q7 — NOT EXISTS: Employees who are the sole member of their department

**Question:** Find employees who are the only person in their department.

```sql
SELECT name, department
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.department = e1.department
      AND e2.id <> e1.id
);
```

**Result:**

| name  | department |
| ----- | ---------- |
| David | Sales      |

> The logical inverse of Q6. `NOT EXISTS` is safe with `NULL` values — unlike `NOT IN`, it will never silently return an empty result set due to a `NULL` in the subquery.

---

## Q8 — Scalar Subquery in SELECT: Each employee alongside their department average

**Question:** Show every employee's name, salary, and their department's average salary in the same row.

```sql
SELECT
    name,
    department,
    salary,
    (
        SELECT ROUND(AVG(salary), 0)
        FROM employees e2
        WHERE e2.department = e1.department
    ) AS dept_avg_salary,
    salary - (
        SELECT ROUND(AVG(salary), 0)
        FROM employees e2
        WHERE e2.department = e1.department
    ) AS diff_from_avg
FROM employees e1
ORDER BY department, salary DESC;
```

**Result:**

| name  | department | salary | dept_avg_salary | diff_from_avg |
| ----- | ---------- | -----: | --------------: | ------------: |
| Anna  | HR         | 75,000 |          67,500 |        +7,500 |
| Mike  | HR         | 60,000 |          67,500 |        -7,500 |
| Sarah | IT         | 90,000 |          83,333 |        +6,667 |
| Lisa  | IT         | 90,000 |          83,333 |        +6,667 |
| John  | IT         | 70,000 |          83,333 |       -13,333 |
| David | Sales      | 80,000 |          80,000 |             0 |

> A scalar subquery in `SELECT` runs once per row. For better performance on large tables, replace with a `JOIN` to a pre-aggregated CTE (shown in the Theory section below).

---

---

# 🔹 Part 2 — Window Functions

---

## 📖 Theory: What are Window Functions?

A **window function** performs a calculation across a set of rows that are **related to the current row** — without collapsing those rows into a single output row the way `GROUP BY` does.

```
GROUP BY  → collapses N rows into 1 row per group
Window    → keeps all N rows, adds a new calculated column
```

### Full Syntax

```sql
function_name(expression)
OVER (
    [PARTITION BY partition_col, ...]   -- divide into groups (optional)
    [ORDER BY sort_col [ASC|DESC], ...]  -- order within each partition
    [ROWS | RANGE frame_clause]          -- window frame (optional)
)
```

### The Three Clauses Inside OVER

| Clause | Purpose | Optional? |
|--------|---------|-----------|
| `PARTITION BY` | Splits rows into independent groups; function resets per group | Yes |
| `ORDER BY` | Defines row order within each partition | Required for ranking / running totals |
| Frame clause | Controls which rows around the current row are included | Yes (has default) |

### Categories of Window Functions

| Category | Functions | Use Case |
|----------|-----------|----------|
| **Ranking** | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()` | Rank rows, top-N, leaderboards |
| **Offset** | `LAG()`, `LEAD()`, `FIRST_VALUE()`, `LAST_VALUE()` | Compare with previous/next rows |
| **Aggregate** | `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()` | Running totals, partition totals, moving averages |

### ROW_NUMBER vs RANK vs DENSE_RANK (Key Differences)

Given salaries: **90, 90, 80, 75, 70, 60**

| Employee | Salary | ROW_NUMBER | RANK | DENSE_RANK |
|----------|-------:| ----------:| ----:| ----------:|
| Sarah    | 90,000 | 1          | 1    | 1          |
| Lisa     | 90,000 | 2          | 1    | 1          |
| David    | 80,000 | 3          | 3    | 2          |
| Anna     | 75,000 | 4          | 4    | 3          |
| John     | 70,000 | 5          | 5    | 4          |
| Mike     | 60,000 | 6          | 6    | 5          |

- `ROW_NUMBER` — always unique; arbitrary for ties (order depends on engine).
- `RANK` — tied rows get the **same rank**; next rank **skips** (gap of 2 after a tie of 2).
- `DENSE_RANK` — tied rows get the **same rank**; next rank is **consecutive** (no gap).

### Frame Clause (Common Patterns)

```sql
-- Default for ORDER BY (RANGE): includes all rows with same ORDER BY value
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- Explicit running total (row-by-row, no tie ambiguity):
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- Moving 3-row average (current + 2 preceding):
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

### Important Rule

> You **cannot** use a window function alias in the same query's `WHERE` clause — window functions are evaluated during `SELECT`, which runs after `WHERE`. Always wrap in a subquery or CTE to filter on window results.

```sql
-- WRONG: rn is not visible in WHERE of the same query
SELECT *, ROW_NUMBER() OVER(...) AS rn FROM employees WHERE rn = 1;

-- CORRECT: wrap in a subquery
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(...) AS rn FROM employees
) t
WHERE t.rn = 1;
```

---

## Q9 — ROW_NUMBER(): Rank employees within each department by salary

**Question:** Assign a rank to each employee within their department, ordered from highest to lowest salary.

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC, id ASC    -- id as tie-breaker for determinism
    ) AS dept_rank
FROM employees
ORDER BY department, dept_rank;
```

**Result:**

| name  | department | salary | dept_rank |
| ----- | ---------- | -----: | --------: |
| Anna  | HR         | 75,000 |         1 |
| Mike  | HR         | 60,000 |         2 |
| Sarah | IT         | 90,000 |         1 |
| Lisa  | IT         | 90,000 |         2 |
| John  | IT         | 70,000 |         3 |
| David | Sales      | 80,000 |         1 |

> `PARTITION BY department` resets the counter at the start of each department. Without it, ROW_NUMBER would count across the entire table as one group.

---

## Q10 — ROW_NUMBER() Top-N per Group: Top 2 salaries per department

**Question:** Find the top 2 highest-paid employees in each department.

```sql
SELECT name, department, salary, dept_rank
FROM (
    SELECT
        name, department, salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, id ASC
        ) AS dept_rank
    FROM employees
) ranked
WHERE dept_rank <= 2
ORDER BY department, dept_rank;
```

**Result:**

| name  | department | salary | dept_rank |
| ----- | ---------- | -----: | --------: |
| Anna  | HR         | 75,000 |         1 |
| Mike  | HR         | 60,000 |         2 |
| Sarah | IT         | 90,000 |         1 |
| Lisa  | IT         | 90,000 |         2 |
| David | Sales      | 80,000 |         1 |

> John (IT, rank 3) is excluded. For "Top-N per group" always use `ROW_NUMBER()` — it always returns exactly N rows per group. Using `RANK()` might give more than N rows when there are ties at the boundary.

---

## Q11 — RANK(): Global salary ranking with gaps on ties

**Question:** Rank all employees globally by salary (highest first). Show how `RANK()` skips numbers when two employees share the same salary.

```sql
SELECT
    name,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees
ORDER BY salary_rank;
```

**Result:**

| name  | department | salary | salary_rank |
| ----- | ---------- | -----: | ----------: |
| Sarah | IT         | 90,000 |           1 |
| Lisa  | IT         | 90,000 |           1 |
| David | Sales      | 80,000 |           3 |
| Anna  | HR         | 75,000 |           4 |
| John  | IT         | 70,000 |           5 |
| Mike  | HR         | 60,000 |           6 |

> Sarah and Lisa both get rank **1**. The next rank is **3** (rank 2 is skipped because two rows occupy positions 1 and 2). This "gap" is the key characteristic of `RANK()`.

---

## Q12 — DENSE_RANK(): Global salary ranking without gaps

**Question:** Rank all employees globally by salary. Show how `DENSE_RANK()` keeps consecutive numbers even when there are ties.

```sql
SELECT
    name,
    department,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees
ORDER BY salary_rank;
```

**Result:**

| name  | department | salary | salary_rank |
| ----- | ---------- | -----: | ----------: |
| Sarah | IT         | 90,000 |           1 |
| Lisa  | IT         | 90,000 |           1 |
| David | Sales      | 80,000 |           2 |
| Anna  | HR         | 75,000 |           3 |
| John  | IT         | 70,000 |           4 |
| Mike  | HR         | 60,000 |           5 |

> David gets rank **2** (not 3). `DENSE_RANK()` never skips a number — this is why it is preferred for "Find the Nth highest salary" problems.

---

## Q13 — RANK vs DENSE_RANK: Side-by-side comparison

**Question:** Show both `RANK()` and `DENSE_RANK()` in the same query so the difference is visible.

```sql
SELECT
    name,
    salary,
    RANK()       OVER (ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk,
    ROW_NUMBER() OVER (ORDER BY salary DESC, id ASC) AS row_num
FROM employees
ORDER BY salary DESC, id ASC;
```

**Result:**

| name  | salary | rnk | dense_rnk | row_num |
| ----- | -----: | --: | --------: | ------: |
| Sarah | 90,000 |   1 |         1 |       1 |
| Lisa  | 90,000 |   1 |         1 |       2 |
| David | 80,000 |   3 |         2 |       3 |
| Anna  | 75,000 |   4 |         3 |       4 |
| John  | 70,000 |   5 |         4 |       5 |
| Mike  | 60,000 |   6 |         5 |       6 |

> **Interview answer:** "RANK skips the next number after a tie (gap). DENSE_RANK does not skip. ROW_NUMBER is always unique regardless of ties."

---

## Q14 — DENSE_RANK(): Find the second highest salary

**Question:** Find the employee(s) with the second highest distinct salary.

```sql
SELECT name, department, salary
FROM (
    SELECT
        name, department, salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 2;
```

**Result:**

| name  | department | salary |
| ----- | ---------- | -----: |
| David | Sales      | 80,000 |

> Using `DENSE_RANK()` guarantees that "second highest" means the second distinct salary value. Change `rnk = 2` to `rnk = 3` for the third highest, etc.

---

## Q15 — DENSE_RANK(): Nth highest salary (parameterised)

**Question:** Write a reusable query to find the Nth highest distinct salary. Find the 3rd highest.

```sql
SELECT name, department, salary
FROM (
    SELECT
        name, department, salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 3;          -- change this number for any N
```

**Result:** *(3rd highest distinct salary = 75,000)*

| name | department | salary |
| ---- | ---------- | -----: |
| Anna | HR         | 75,000 |

**Salary rank map for reference:**

| Dense Rank | Salary | Employees |
| ---------: | -----: | --------- |
| 1 | 90,000 | Sarah, Lisa |
| 2 | 80,000 | David |
| 3 | 75,000 | Anna |
| 4 | 70,000 | John |
| 5 | 60,000 | Mike |

---

## Q16 — LAG(): Compare each employee's salary to the previous employee

**Question:** Show each employee alongside the salary of the previous employee (when sorted by salary ascending). Useful for spotting salary jumps.

```sql
SELECT
    name,
    salary,
    LAG(salary)  OVER (ORDER BY salary ASC, id ASC) AS prev_salary,
    salary - LAG(salary) OVER (ORDER BY salary ASC, id ASC) AS increase
FROM employees
ORDER BY salary ASC, id ASC;
```

**Result:**

| name  | salary | prev_salary | increase |
| ----- | -----: | ----------: | -------: |
| Mike  | 60,000 |        NULL |     NULL |
| John  | 70,000 |      60,000 |   10,000 |
| Anna  | 75,000 |      70,000 |    5,000 |
| David | 80,000 |      75,000 |    5,000 |
| Sarah | 90,000 |      80,000 |   10,000 |
| Lisa  | 90,000 |      90,000 |        0 |

> The first row (Mike) has no previous row, so `LAG()` returns `NULL`. `LAG(col, 2)` looks back 2 rows; `LAG(col, 1, 0)` uses `0` as the default instead of `NULL`.

---

## Q17 — LEAD(): Compare each employee's salary to the next employee

**Question:** Show each employee alongside the salary of the next employee (sorted by salary ascending). Useful for identifying how far an employee is from the next pay bracket.

```sql
SELECT
    name,
    salary,
    LEAD(salary) OVER (ORDER BY salary ASC, id ASC) AS next_salary,
    LEAD(salary) OVER (ORDER BY salary ASC, id ASC) - salary AS gap_to_next
FROM employees
ORDER BY salary ASC, id ASC;
```

**Result:**

| name  | salary | next_salary | gap_to_next |
| ----- | -----: | ----------: | ----------: |
| Mike  | 60,000 |      70,000 |      10,000 |
| John  | 70,000 |      75,000 |       5,000 |
| Anna  | 75,000 |      80,000 |       5,000 |
| David | 80,000 |      90,000 |      10,000 |
| Sarah | 90,000 |      90,000 |           0 |
| Lisa  | 90,000 |        NULL |        NULL |

> Lisa is the last row (no next), so `LEAD()` returns `NULL`. Use `LEAD(col, 1, 0)` to default to `0` instead of `NULL`.

---

## Q18 — SUM OVER: Global running salary total

**Question:** Show a running total of salary as you move through all employees ordered by salary ascending.

```sql
SELECT
    name,
    salary,
    SUM(salary) OVER (
        ORDER BY salary ASC, id ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM employees
ORDER BY salary ASC, id ASC;
```

**Result:**

| name  | salary | running_total |
| ----- | -----: | ------------: |
| Mike  | 60,000 |        60,000 |
| John  | 70,000 |       130,000 |
| Anna  | 75,000 |       205,000 |
| David | 80,000 |       285,000 |
| Sarah | 90,000 |       375,000 |
| Lisa  | 90,000 |       465,000 |

> `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` includes every row from the start up to and including the current row. This is explicit and avoids tie ambiguity. Without specifying `ROWS`, MySQL uses `RANGE`, which sums **all tied rows together** (Sarah and Lisa would both show 465,000).

---

## Q19 — SUM OVER with PARTITION: Running total within each department

**Question:** Show a running salary total that resets at the start of each department.

```sql
SELECT
    name,
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY salary ASC, id ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS dept_running_total
FROM employees
ORDER BY department, salary ASC;
```

**Result:**

| name  | department | salary | dept_running_total |
| ----- | ---------- | -----: | -----------------: |
| Mike  | HR         | 60,000 |             60,000 |
| Anna  | HR         | 75,000 |            135,000 |
| John  | IT         | 70,000 |             70,000 |
| Sarah | IT         | 90,000 |            160,000 |
| Lisa  | IT         | 90,000 |            250,000 |
| David | Sales      | 80,000 |             80,000 |

> The running total resets to zero at the start of HR, then IT, then Sales. `PARTITION BY` is the key — without it, the running total would be global (Q18).

---

## Q20 — SUM OVER PARTITION (no ORDER BY): Department total on every row

**Question:** Show each employee's details alongside the total salary budget for their entire department (not running, the full total).

```sql
SELECT
    name,
    department,
    salary,
    SUM(salary)  OVER (PARTITION BY department) AS dept_total,
    COUNT(*)     OVER (PARTITION BY department) AS dept_headcount,
    ROUND(AVG(salary) OVER (PARTITION BY department), 0) AS dept_avg
FROM employees
ORDER BY department, salary DESC;
```

**Result:**

| name  | department | salary | dept_total | dept_headcount | dept_avg |
| ----- | ---------- | -----: | ---------: | -------------: | -------: |
| Anna  | HR         | 75,000 |    135,000 |              2 |   67,500 |
| Mike  | HR         | 60,000 |    135,000 |              2 |   67,500 |
| Sarah | IT         | 90,000 |    250,000 |              3 |   83,333 |
| Lisa  | IT         | 90,000 |    250,000 |              3 |   83,333 |
| John  | IT         | 70,000 |    250,000 |              3 |   83,333 |
| David | Sales      | 80,000 |     80,000 |              1 |   80,000 |

> Without `ORDER BY` inside `OVER`, the window includes **all rows in the partition** — so every employee sees the same department-wide totals. This is very useful for dashboards and reports.

---

---

# 🎯 Part 3 — Theory Interview Q&A

---

## Q21 — What is the difference between a subquery and a JOIN?

**A:**

| | Subquery | JOIN |
|--|---------|------|
| Structure | Query nested inside another | Two inputs combined side-by-side |
| Output | Used as a filter or scalar value | Produces columns from both sides |
| Readability | Intuitive for filtering logic | Better for combining related data |
| Performance | Correlated subqueries can be slow (run per row) | Usually faster; optimizer can exploit indexes on JOIN keys |

```sql
-- Same question, two approaches: employees above average salary

-- Subquery approach
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- JOIN approach (derived table)
SELECT e.name
FROM employees e
JOIN (SELECT AVG(salary) AS avg_sal FROM employees) avg_table
ON e.salary > avg_table.avg_sal;
```

**Rule of thumb:** Use a subquery when filtering against an aggregate or checking existence. Use a JOIN when you need columns from both tables in the result.

---

## Q22 — What is a correlated subquery and what is its performance concern?

**A:** A correlated subquery references a column from the outer query — it is logically re-executed for every row the outer query processes.

```sql
-- Correlated: inner query re-runs for each employee row
SELECT name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) FROM employees e2
    WHERE e2.department = e1.department   -- ← correlation
);
```

**Performance concern:** O(N × M) — for every outer row, the inner query scans M rows. Rewrite using a CTE or derived table to compute the aggregate **once**:

```sql
-- Better: compute all dept averages once
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;
```

---

## Q23 — What is the difference between `EXISTS` and `IN`?

**A:**

| | `IN` | `EXISTS` |
|--|-----|---------|
| How it works | Gets entire subquery result, checks membership | Checks if at least one row matches (short-circuits) |
| NULL behaviour | Dangerous — `NULL` in list can cause zero results | Safe — returns TRUE/FALSE, not affected by NULLs |
| Best for | Small non-nullable subquery results | Large tables, correlated lookups |

```sql
-- IN: fetches all matching dept names first
SELECT name FROM employees
WHERE department IN (SELECT department FROM employees WHERE salary > 80000);

-- EXISTS: stops at first match per outer row
SELECT name FROM employees e1
WHERE EXISTS (
    SELECT 1 FROM employees e2
    WHERE e2.department = e1.department AND e2.salary > 80000
);
```

**When to avoid `NOT IN`:** If the subquery can return `NULL`, `NOT IN` returns zero rows due to SQL's three-valued logic. Use `NOT EXISTS` instead.

---

## Q24 — Window functions vs GROUP BY: What is the difference?

**A:**

| | `GROUP BY` | Window Function |
|--|-----------|----------------|
| Row output | One row per group (collapses data) | Same number of rows as input (no collapse) |
| Access to individual rows | Lost after grouping | Preserved |
| Use for | Aggregation summaries | Per-row analytics, rankings, running totals |

```sql
-- GROUP BY: 3 rows (one per dept)
SELECT department, SUM(salary) AS total
FROM employees
GROUP BY department;

-- Window: 6 rows (every employee + dept total as a column)
SELECT name, department, salary,
       SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;
```

---

## Q25 — Explain PARTITION BY vs ORDER BY inside a window function.

**A:**
- **`PARTITION BY`** — divides the result set into independent groups. The window function resets at the start of each partition. Think of it as the window version of `GROUP BY`.
- **`ORDER BY`** — defines the sequence of rows within each partition. Required for ranking and running totals.

```sql
SELECT
    name, department, salary,
    -- PARTITION BY alone: full-dept total (no ordering needed)
    SUM(salary) OVER (PARTITION BY department) AS dept_total,
    -- PARTITION BY + ORDER BY: running total within dept
    SUM(salary) OVER (PARTITION BY department ORDER BY salary ASC) AS running_total,
    -- ORDER BY alone: global running total
    SUM(salary) OVER (ORDER BY salary ASC) AS global_running_total
FROM employees;
```

---

## Q26 — What is the difference between RANK, DENSE_RANK, and ROW_NUMBER?

**A:**

| Function | Ties | Gaps after tie | Always unique |
|----------|------|----------------|---------------|
| `ROW_NUMBER()` | Arbitrary ordering | No concept of gap | Yes |
| `RANK()` | Same rank for ties | Yes — skips next rank(s) | No |
| `DENSE_RANK()` | Same rank for ties | No — consecutive | No |

**Use:**
- `ROW_NUMBER` → Top-N per group (need exactly N rows)
- `RANK` → Competitive rankings where gaps make sense (sports leaderboard)
- `DENSE_RANK` → Nth highest value problems (no skipped positions)

---

## Q27 — Why can't you use a window function result in the WHERE clause of the same query?

**A:** SQL processes clauses in this logical order:

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

Window functions are computed during the `SELECT` phase. By the time `WHERE` runs, the window function result does not exist yet. Wrap in a subquery or CTE to filter:

```sql
-- WRONG: rn doesn't exist when WHERE executes
SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
FROM employees WHERE rn = 1;

-- CORRECT
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) t
WHERE rn = 1;
```

---

## Q28 — When would you use LAG/LEAD vs a self-join?

**A:**
- **`LAG()` / `LEAD()`** — cleaner, more readable, better performance. Use when you need the previous or next row based on a defined ordering.
- **Self-join** — needed when the "previous" row is determined by a complex condition, or when you need to access more than one prior row with different filtering logic.

```sql
-- LAG: clean, one pass
SELECT name, salary, LAG(salary) OVER (ORDER BY salary) AS prev_salary
FROM employees;

-- Self-join: verbose, two scans
SELECT e1.name, e1.salary, MAX(e2.salary) AS prev_salary
FROM employees e1
LEFT JOIN employees e2 ON e2.salary < e1.salary
GROUP BY e1.name, e1.salary;
```

---

## Q29 — What is a CTE and when do you use it over a subquery?

**A:** A CTE (Common Table Expression) is a named temporary result defined with `WITH` at the top of the query.

```sql
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
)
SELECT e.name, e.salary, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;
```

**Prefer CTE over subquery when:**
- The same derived result is referenced **more than once** (subquery would repeat the calculation).
- The query is deeply nested and readability suffers.
- You need a **recursive query** (hierarchies, trees — only CTEs support `WITH RECURSIVE`).

**Prefer a derived table (inline subquery) when:** it is used only once and keeping it inline makes the logic more obvious.

---

## Q30 — How does the ROWS frame clause affect SUM OVER?

**A:** The frame clause controls which rows around the current row are included in the window calculation.

```sql
-- RANGE (default when ORDER BY present): all rows with same ORDER BY value as current
SUM(salary) OVER (ORDER BY salary)
-- → Sarah and Lisa (both 90k) both see the full total 465k

-- ROWS: physical row position, not value-based
SUM(salary) OVER (ORDER BY salary ASC, id ASC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
-- → Sarah sees 375k, Lisa sees 465k (incremental)

-- Moving 3-row window
AVG(salary) OVER (ORDER BY id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
```

**Interview answer:** "Use `ROWS` when you want exact per-row increments (running total). Use `RANGE` when you want all tied values treated as a single unit."

---

---

# 💡 Part 4 — Real Interview Problems

---

## Q31: Second highest salary (three methods)

**Method A — OFFSET (simple, no ties awareness)**

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

or

select
	e1.salary
from
	employees e1
where
	(
	select
		count(distinct e2.salary)
	from
		employees e2
	where
		e2.salary > e1.salary
) = 1;
```

**Method B — DENSE_RANK (handles ties correctly)**

```sql
SELECT name, salary
FROM (
    SELECT name, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;
```

**Method C — Subquery (classic approach)**

```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```


**Result:** David — 80,000

> **Interview tip:** Always use Method B in interviews — it handles ties and scales to "Nth highest" by changing the number.

---

## Q32: Top-N per group (most asked pattern)

**Question:** Find the top 1 highest-paid employee in each department.

```sql
SELECT name, department, salary
FROM (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (
               PARTITION BY department
               ORDER BY salary DESC, id ASC
           ) AS rn
    FROM employees
) t
WHERE rn = 1
ORDER BY department;

OR

select
	e1.salary
from
	employees e1
where
	(
	select
		count(distinct e2.salary)
	from
		employees e2
	where
		e2.salary > e1.salary
) = N - 1;

```

**Result:**

| name  | department | salary |
| ----- | ---------- | -----: |
| Anna  | HR         | 75,000 |
| Sarah | IT         | 90,000 |
| David | Sales      | 80,000 |

> Use `ROW_NUMBER()` (not `RANK`) so you get **exactly one row per department** even when there are ties. To get all tied top earners per department, use `RANK()` and filter `rn = 1`.

---

## Q33: Duplicate records — detect and delete

**Detect duplicates (same name + dept + salary):**

```sql
SELECT name, department, salary, COUNT(*) AS occurrences
FROM employees
GROUP BY name, department, salary
HAVING COUNT(*) > 1;
```

**Find duplicate row IDs (keep lowest id):**

```sql
SELECT id, name, department, salary, rn
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY name, department, salary
               ORDER BY id ASC
           ) AS rn
    FROM employees
) t
WHERE rn > 1;    -- rn = 1 is the "original"; rn > 1 are duplicates
```

**Delete duplicates, keep the first occurrence:**

```sql
DELETE FROM employees
WHERE id IN (
    SELECT id FROM (
        SELECT id,
               ROW_NUMBER() OVER (
                   PARTITION BY name, department, salary
                   ORDER BY id ASC
               ) AS rn
        FROM employees
    ) t
    WHERE rn > 1
);
```

---

## Q34: Running salary comparison — employees who earn more than previous

**Question:** Find employees whose salary is higher than the previous employee's salary (sorted by id).

```sql
SELECT name, salary, prev_salary
FROM (
    SELECT
        name, salary,
        LAG(salary) OVER (ORDER BY id ASC) AS prev_salary
    FROM employees
) t
WHERE prev_salary IS NOT NULL
  AND salary > prev_salary
ORDER BY id;
```

**Result:**

| name  | salary | prev_salary |
| ----- | -----: | ----------: |
| Sarah | 90,000 |      70,000 |
| Anna  | 75,000 |      60,000 |
| David | 80,000 |      75,000 |
| Lisa  | 90,000 |      80,000 |

 
## Q35 — Find employees whose salary is above the overall average. Show their salary and the average.**

```sql
SELECT name, salary,
       ROUND((SELECT AVG(salary) FROM employees), 0) AS company_avg
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

## Q36 — Highest salary per department (using window functions)**

```sql
SELECT DISTINCT department,
       MAX(salary) OVER (PARTITION BY department) AS max_salary
FROM employees
ORDER BY department;
```

---

## Q37 — 3rd highest distinct salary**

```sql
SELECT name, salary
FROM (
    SELECT name, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 3;
-- Result: Anna (75,000)
```

---

## Q38 — Running total per department ordered by salary**

```sql
SELECT name, department, salary,
       SUM(salary) OVER (
           PARTITION BY department
           ORDER BY salary ASC, id ASC
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM employees
ORDER BY department, salary ASC;
```

---

## Q39 — Find all departments where average salary > 70,000**

```sql
SELECT department, ROUND(AVG(salary), 0) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
-- Result: IT (83,333), Sales (80,000)
```

---

## Q40 — For each employee show their salary percentile rank (0 to 1)**

```sql
SELECT name, salary,
       ROUND(PERCENT_RANK() OVER (ORDER BY salary ASC), 2) AS percentile
FROM employees
ORDER BY salary ASC;
```

---

## Q41 — Divide employees into 3 salary buckets (low / mid / high)**

```sql
SELECT name, salary,
       NTILE(3) OVER (ORDER BY salary ASC) AS bucket,
       CASE NTILE(3) OVER (ORDER BY salary ASC)
           WHEN 1 THEN 'Low'
           WHEN 2 THEN 'Mid'
           WHEN 3 THEN 'High'
       END AS salary_band
FROM employees
ORDER BY salary ASC;
```

---


# 🧠 Key Takeaways

| Concept | Best Used For |
|---------|--------------|
| **Scalar Subquery** | Filtering against a single computed value (AVG, MAX) |
| **IN Subquery** | Filtering against a list of values from another table |
| **Correlated Subquery** | Per-row filtering against a related aggregate |
| **EXISTS / NOT EXISTS** | Checking whether related rows exist (safe with NULLs) |
| **ROW_NUMBER()** | Top-N per group (exactly N rows, no ties) |
| **RANK()** | Competitive ranking with gaps (sports, leaderboards) |
| **DENSE_RANK()** | Nth highest value (no gaps, handles ties cleanly) |
| **LAG() / LEAD()** | Comparing a row to its predecessor or successor |
| **SUM OVER PARTITION** | Adding a group-level aggregate to each row without GROUP BY |
| **CTE (WITH)** | Naming and reusing intermediate results; recursive queries |

---
