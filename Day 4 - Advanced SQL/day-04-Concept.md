# 📘 Day 4 — Advanced SQL (Subqueries + Window Functions)

This module focuses on **advanced SQL concepts** that are frequently asked in backend and product-based company interviews.

---

# 📌 Topics Covered

* Subqueries (Scalar, Correlated, Nested)
* Window Functions (Ranking, Analytics)
* Real Interview Problems
* Practice Questions

---

# 🧱 Sample Dataset

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary INT
);

INSERT INTO employees VALUES
(1, 'John', 'IT', 70000),
(2, 'Sarah', 'IT', 90000),
(3, 'Mike', 'HR', 60000),
(4, 'Anna', 'HR', 75000),
(5, 'David', 'Sales', 80000);
```

### Employees (Sample Records)

| id | name  | department | salary |
| ---: | --- | --- | ---: |
| 1 | John  | IT     | 70000 |
| 2 | Sarah | IT     | 90000 |
| 3 | Mike  | HR     | 60000 |
| 4 | Anna  | HR     | 75000 |
| 5 | David | Sales  | 80000 |

---

# 🔹 Part 1: Subqueries

## 📌 What is a Subquery?

A query inside another SQL query.

## 🔸 Types

| Type       | Description           |
| ---------- | --------------------- |
| Scalar     | Returns single value  |
| Column     | Returns single column |
| Row        | Returns single row    |
| Correlated | Executes per row      |

---

## ✅ 1. Employees earning more than average salary

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) FROM employees
);
```

**Output (expected):**

| name  | salary |
| --- | ---: |
| Sarah | 90000 |
| David | 80000 |

---

## ✅ 2. Employees with highest salary

```sql
SELECT *
FROM employees
WHERE salary = (
    SELECT MAX(salary) FROM employees
);
```

**Output (expected):**

| id | name  | department | salary |
| ---: | --- | --- | ---: |
| 2 | Sarah | IT | 90000 |

---

## ✅ 3. Employees in same department as John

```sql
SELECT *
FROM employees
WHERE department = (
    SELECT department FROM employees WHERE name = 'John'
);
```

**Output (expected):**

| id | name  | department | salary |
| ---: | --- | --- | ---: |
| 1 | John  | IT | 70000 |
| 2 | Sarah | IT | 90000 |

---

## ✅ 4. Departments with more than 1 employee

```sql
SELECT *
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 1
);
```

**Output (expected):**

| id | name  | department | salary |
| ---: | --- | --- | ---: |
| 1 | John | IT | 70000 |
| 2 | Sarah | IT | 90000 |
| 3 | Mike | HR | 60000 |
| 4 | Anna | HR | 75000 |

---

## ✅ 5. Correlated Subquery (Important)

```sql
SELECT name, department, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e1.department = e2.department
);
```

**Output (expected):**

| name  | department | salary |
| --- | --- | ---: |
| Sarah | IT | 90000 |
| Anna  | HR | 75000 |

---

# 🔹 Part 2: Window Functions

## 📌 What are Window Functions?

Perform calculations across rows **without grouping them**.

---

## 🔸 Syntax

```sql
FUNCTION() OVER (
    PARTITION BY column
    ORDER BY column
)
```

---

## ✅ 6. ROW_NUMBER()

```sql
SELECT name, department, salary,
       ROW_NUMBER() OVER(
           PARTITION BY department
           ORDER BY salary DESC
       ) AS row_num
FROM employees;
```

**Output (expected):**

| name  | department | salary | row_num |
| --- | --- | ---: | ---: |
| John  | IT | 70000 | 2 |
| Sarah | IT | 90000 | 1 |
| Mike  | HR | 60000 | 2 |
| Anna  | HR | 75000 | 1 |
| David | Sales | 80000 | 1 |

---

## ✅ 7. Top 2 highest salary per department

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employees
) t
WHERE rn <= 2;
```

**Output (expected):**

| id | name  | department | salary | rn |
| ---: | --- | --- | ---: | ---: |
| 1 | John  | IT    | 70000 | 2 |
| 2 | Sarah | IT    | 90000 | 1 |
| 3 | Mike  | HR    | 60000 | 2 |
| 4 | Anna  | HR    | 75000 | 1 |
| 5 | David | Sales | 80000 | 1 |

---

## ✅ 8. RANK vs DENSE_RANK

```sql
SELECT name, salary,
       RANK() OVER(ORDER BY salary DESC) rank_val,
       DENSE_RANK() OVER(ORDER BY salary DESC) dense_rank_val
FROM employees;
```

**Output (expected):**

| name  | salary | rank_val | dense_rank_val |
| --- | ---: | ---: | ---: |
| John  | 70000 | 4 | 4 |
| Sarah | 90000 | 1 | 1 |
| Mike  | 60000 | 5 | 5 |
| Anna  | 75000 | 3 | 3 |
| David | 80000 | 2 | 2 |

---

## ✅ 9. LAG() — Previous Row

```sql
SELECT name, salary,
       LAG(salary) OVER(ORDER BY salary) AS previous_salary
FROM employees;
```

**Output (expected):**

| name  | salary | previous_salary |
| --- | ---: | ---: |
| John  | 70000 | 60000 |
| Sarah | 90000 | 80000 |
| Mike  | 60000 | NULL |
| Anna  | 75000 | 70000 |
| David | 80000 | 75000 |

---

## ✅ 10. LEAD() — Next Row

```sql
SELECT name, salary,
       LEAD(salary) OVER(ORDER BY salary) AS next_salary
FROM employees;
```

**Output (expected):**

| name  | salary | next_salary |
| --- | ---: | ---: |
| John  | 70000 | 75000 |
| Sarah | 90000 | NULL |
| Mike  | 60000 | 70000 |
| Anna  | 75000 | 80000 |
| David | 80000 | 90000 |

---

## ✅ 11. Running Total

```sql
SELECT name, salary,
       SUM(salary) OVER(ORDER BY salary) AS running_total
FROM employees;
```

**Output (expected):**

| name  | salary | running_total |
| --- | ---: | ---: |
| John  | 70000 | 130000 |
| Sarah | 90000 | 375000 |
| Mike  | 60000 | 60000 |
| Anna  | 75000 | 205000 |
| David | 80000 | 285000 |

---

## ✅ 12. Department Salary Total

```sql
SELECT name, department, salary,
       SUM(salary) OVER(PARTITION BY department) AS dept_total
FROM employees;
```

**Output (expected):**

| name  | department | salary | dept_total |
| --- | --- | ---: | ---: |
| John  | IT | 70000 | 160000 |
| Sarah | IT | 90000 | 160000 |
| Mike  | HR | 60000 | 135000 |
| Anna  | HR | 75000 | 135000 |
| David | Sales | 80000 | 80000 |

---

# 🎯 Interview Questions

## 1. Difference between subquery and join
### Subquery
A **subquery** is a query inside another query. It’s commonly used to compute a scalar value (like `AVG`, `MAX`) or to produce a set used by `IN`, `EXISTS`, etc.

Example (employees earning more than overall average salary):
```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### Join
A **join** combines rows from two inputs (tables or derived tables) based on a condition.

Same logic using a join with a derived table (avg salary):
```sql
SELECT e.name, e.salary
FROM employees e
JOIN (
    SELECT AVG(salary) AS avg_salary
    FROM employees
) a
ON e.salary > a.avg_salary;
```

How to choose:
- Use a **subquery** when you need computed/nested filtering logic.
- Use a **join** when the logic naturally combines row sets.

## 2. What is correlated subquery?
A **correlated subquery** references columns from the outer query, so it is evaluated conceptually **once per outer row**.

Example (salary greater than average of the same department):
```sql
SELECT name, department, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e1.department = e2.department
);
```

The correlation is: `e1.department = e2.department`.

## 3. Window function vs GROUP BY
### GROUP BY
`GROUP BY` collapses rows into one row per group.

Example: total salary per department:
```sql
SELECT department, SUM(salary) AS dept_total
FROM employees
GROUP BY department;
```

### Window functions
Window functions keep row-level detail and add analytical columns.

Example: department total repeated on every row:
```sql
SELECT name, department, salary,
       SUM(salary) OVER (PARTITION BY department) AS dept_total
FROM employees;
```

Rule of thumb:
- `GROUP BY` => fewer rows (aggregation)
- Window => same rows + analytics

## 4. Difference between RANK and DENSE_RANK
Both are used like: `RANK() OVER (ORDER BY ...)` and assign rank numbers within a window.

- `RANK()` skips numbers when ties exist.
- `DENSE_RANK()` does not skip numbers (no gaps).

Pattern:
```sql
SELECT name, salary,
       RANK() OVER(ORDER BY salary DESC) AS rank_val,
       DENSE_RANK() OVER(ORDER BY salary DESC) AS dense_rank_val
FROM employees;
```

In your sample data salaries are distinct, so both behave the same. The gap-vs-no-gap difference shows up when there are ties on `salary`.

## 5. What is PARTITION BY?
`PARTITION BY` divides rows into partitions. Window calculations are applied separately per partition.

Example: top 2 salaries per department (top-N per group):
```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employees
) t
WHERE rn <= 2;
```

Without `PARTITION BY`, ranking would be computed across the entire table as one partition.

## 6. How to find Nth highest salary?
Use ranking functions (window functions).

### Nth highest distinct salary
Use `DENSE_RANK()`:
```sql
SELECT name, salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 3; -- 3rd highest distinct salary
```

### Nth row in sorted order (ties included)
Use `ROW_NUMBER()` and add a deterministic tie-breaker:
```sql
SELECT name, salary
FROM (
    SELECT *,
           ROW_NUMBER() OVER(ORDER BY salary DESC, id DESC) AS rn
    FROM employees
) t
WHERE rn = 3;
```

## 7. How to get top N per group?
Top-N per group means: “for each group key, take top N rows by some metric”.

Use `ROW_NUMBER()` with `PARTITION BY` and filter `rn <= N` outside:
```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employees
) t
WHERE rn <= 2;
```

## 8. Explain ROW_NUMBER()
`ROW_NUMBER()` assigns a unique sequential number to each row within a window partition.

Example:
```sql
SELECT name, department, salary,
       ROW_NUMBER() OVER(
         PARTITION BY department
         ORDER BY salary DESC
       ) AS row_num
FROM employees;
```

Note: if ordering columns tie, add tie-breakers (like `id`) for deterministic results.

## 9. Explain LAG and LEAD
`LAG()` returns a value from a previous row; `LEAD()` returns a value from a next row.

Example:
```sql
SELECT name, salary,
       LAG(salary)  OVER(ORDER BY salary) AS previous_salary,
       LEAD(salary) OVER(ORDER BY salary) AS next_salary
FROM employees;
```

`NULL` appears when there is no previous/next row.

## 10. Execution order of window functions
Practical mental model:
1. `FROM` / `JOIN` forms the row set
2. `WHERE` filters rows
3. `SELECT` computes expressions, including window functions
4. Outer query `WHERE`/`HAVING` filters again after window results (if you use a subquery/CTE)
5. Final `ORDER BY` sorts output

Gotcha: you generally can’t filter on a window function result alias in the same query’s `WHERE`. Use a subquery:
```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employees
) t
WHERE rn <= 2;
```

---

# 💡 Real Interview Problems

## 🔸 Find 2nd highest salary

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

### Using Window Function

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER(ORDER BY salary DESC) rnk
    FROM employees
) t
WHERE rnk = 2;
```

---

# 🚀 Practice Questions

1. Find 3rd highest salary

```sql
-- 3rd highest DISTINCT salary (uses DENSE_RANK)
SELECT DISTINCT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER(ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 3;
```

2. Highest salary per department

```sql
-- Top 1 salary in each department (use ROW_NUMBER)
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC, id DESC
           ) AS rn
    FROM employees
) t
WHERE rn = 1;
```

3. Salary greater than previous employee

```sql
-- "Previous employee" is defined by the ordering in the window (here: by salary asc)
SELECT *
FROM (
    SELECT e.*,
           LAG(salary) OVER(ORDER BY salary ASC, id ASC) AS prev_salary
    FROM employees e
) t
WHERE prev_salary IS NOT NULL
  AND salary > prev_salary;
```

4. Running total per department

```sql
-- Running total within each department ordered by salary
SELECT name, department, salary,
       SUM(salary) OVER(
           PARTITION BY department
           ORDER BY salary ASC, id ASC
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM employees;
```

5. Find duplicate records

```sql
-- Duplicate employees by (name, department, salary)
SELECT *
FROM (
    SELECT e.*,
           ROW_NUMBER() OVER(
               PARTITION BY name, department, salary
               ORDER BY id
           ) AS rn
    FROM employees e
) t
WHERE rn > 1;

-- Alternative (duplicates count):
-- SELECT name, department, salary, COUNT(*) AS dup_count
-- FROM employees
-- GROUP BY name, department, salary
-- HAVING COUNT(*) > 1;
```

---

# 🧠 Key Takeaways

* Subqueries are used for **filtering and nested logic**
* Correlated subqueries execute **row-by-row**
* Window functions are used for **analytics and ranking**
* ROW_NUMBER is best for **Top-N problems**
* DENSE_RANK is best for **Nth highest salary**

---

# 📦 Use Cases in Backend Systems

* Leaderboards
* Analytics dashboards
* Financial reports
* Pagination queries
* Deduplication
* Time-series analysis

---

🔥 This is one of the **most important SQL topics for product-based interviews**.

Make sure you practice these patterns thoroughly.
