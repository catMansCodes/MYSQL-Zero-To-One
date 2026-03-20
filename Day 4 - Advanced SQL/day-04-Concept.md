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

---

## ✅ 2. Employees with highest salary

```sql
SELECT *
FROM employees
WHERE salary = (
    SELECT MAX(salary) FROM employees
);
```

---

## ✅ 3. Employees in same department as John

```sql
SELECT *
FROM employees
WHERE department = (
    SELECT department FROM employees WHERE name = 'John'
);
```

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

---

## ✅ 8. RANK vs DENSE_RANK

```sql
SELECT name, salary,
       RANK() OVER(ORDER BY salary DESC) rank_val,
       DENSE_RANK() OVER(ORDER BY salary DESC) dense_rank_val
FROM employees;
```

---

## ✅ 9. LAG() — Previous Row

```sql
SELECT name, salary,
       LAG(salary) OVER(ORDER BY salary) AS previous_salary
FROM employees;
```

---

## ✅ 10. LEAD() — Next Row

```sql
SELECT name, salary,
       LEAD(salary) OVER(ORDER BY salary) AS next_salary
FROM employees;
```

---

## ✅ 11. Running Total

```sql
SELECT name, salary,
       SUM(salary) OVER(ORDER BY salary) AS running_total
FROM employees;
```

---

## ✅ 12. Department Salary Total

```sql
SELECT name, department, salary,
       SUM(salary) OVER(PARTITION BY department) AS dept_total
FROM employees;
```

---

# 🎯 Interview Questions

1. Difference between subquery and join
2. What is correlated subquery?
3. Window function vs GROUP BY
4. Difference between RANK and DENSE_RANK
5. What is PARTITION BY?
6. How to find Nth highest salary?
7. How to get top N per group?
8. Explain ROW_NUMBER()
9. Explain LAG and LEAD
10. Execution order of window functions

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
2. Highest salary per department
3. Salary greater than previous employee
4. Running total per department
5. Find duplicate records

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
