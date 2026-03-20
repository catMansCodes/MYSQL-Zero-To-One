1-Hour SQL Interview Cheatsheet (High-Impact Revision)

This is your last-minute revision sheet before an interview.
Focus: Speed + Recall + Accuracy

⚡ 1. SQL Execution Order (🔥 Very Important)
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

👉 Interview Trick Question:

You cannot use alias in WHERE

You can use alias in ORDER BY

⚡ 2. Most Important Query Templates

✅ 1. SELECT + FILTER
```
SELECT column1, column2
FROM table
WHERE condition;
```
✅ 2. GROUP BY + HAVING
```
SELECT column, COUNT(_)
FROM table
GROUP BY column
HAVING COUNT(_) > 5;
```
✅ 3. JOIN Template
```
SELECT a.col, b.col
FROM table1 a
JOIN table2 b
ON a.id = b.id;
```
✅ 4. LEFT JOIN (Missing Data)
```
SELECT a._
FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
WHERE b.id IS NULL;
```
✅ 5. Subquery Template
```
SELECT _
FROM table
WHERE column > (SELECT AVG(column) FROM table);
```

✅ 6. Window Function Template
```
SELECT _,
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) rn
FROM employees;
```
---
## 3. Top Patterns (🔥 MUST REMEMBER)

### 🧠 1. Second Highest Salary
```
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```
### 🧠 2. Top N per Group
```
SELECT _
FROM (
SELECT _, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) rn
FROM employees
) t
WHERE rn <= 3;
```

### 🧠 3. Find Duplicates
```
SELECT column, COUNT(_)
FROM table
GROUP BY column
HAVING COUNT(\*) > 1;
```
### 🧠 4. Delete Duplicates
```
DELETE t1
FROM table t1
JOIN table t2
ON t1.col = t2.col AND t1.id > t2.id;
```
### 🧠 5. Running Total
```
SELECT id, SUM(value) OVER (ORDER BY id)
FROM table;
```
### 🧠 6. Employees > Manager Salary
```
SELECT e.name
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

## ⚡ 4. Important Functions (Rapid Recall)
```
🔹 Aggregates
COUNT()
SUM()
AVG()
MAX()
MIN()

🔹 String

CONCAT()
SUBSTRING()
```

### 1-Hour SQL Interview Cheatsheet (High-Impact Revision)

- This is your last-minute revision sheet before an interview.

- Focus: Speed + Recall + Accuracy

## ⚡ 1. SQL Execution Order (🔥 Very Important)

`FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`

👉 Interview Trick Question:

- You cannot use alias in `WHERE`
- You can use alias in `ORDER BY`

## ⚡ 2. Most Important Query Templates

**1. SELECT + FILTER**

```sql
SELECT column1, column2
FROM table
WHERE condition;
```

**2. GROUP BY + HAVING**

```sql
SELECT column, COUNT(*)
FROM table
GROUP BY column
HAVING COUNT(*) > 5;
```

**3. JOIN Template**

```sql
SELECT a.col, b.col
FROM table1 a
JOIN table2 b
ON a.id = b.id;
```

**4. LEFT JOIN (Missing Data)**

```sql
SELECT a.*
FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
WHERE b.id IS NULL;
```

**5. Subquery Template**

```sql
SELECT *
FROM table
WHERE column > (SELECT AVG(column) FROM table);
```

**6. Window Function Template**

```sql
SELECT *,
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) rn
FROM employees;
```

## ⚡ 3. Top Patterns (🔥 MUST REMEMBER)

**1. Second Highest Salary**

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**2. Top N per Group**

```sql
SELECT *
FROM (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) rn
        FROM employees
) t
WHERE rn <= 3;
```

**3. Find Duplicates**

```sql
SELECT column, COUNT(*)
FROM table
GROUP BY column
HAVING COUNT(*) > 1;
```

**4. Delete Duplicates**

```sql
DELETE t1
FROM table t1
JOIN table t2
ON t1.col = t2.col AND t1.id > t2.id;
```

**5. Running Total**

```sql
SELECT id, SUM(value) OVER (ORDER BY id)
FROM table;
```

**6. Employees > Manager Salary**

```sql
SELECT e.name
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

## ⚡ 4. Important Functions (Rapid Recall)

**Aggregates**

- `COUNT()`
- `SUM()`
- `AVG()`
- `MAX()`
- `MIN()`

**String**

- `CONCAT()`
- `SUBSTRING()`
- `LENGTH()`

**NULL Handling**

- `IFNULL(column, 0)`
- `COALESCE(col1, col2, 0)`

**Date**

- `NOW()`
- `CURDATE()`
- `DATEDIFF()`

## ⚡ 5. JOIN Quick Difference

| JOIN         | Output               |
| ------------ | -------------------- |
| `INNER JOIN` | Matching rows only   |
| `LEFT JOIN`  | All left + matching  |
| `RIGHT JOIN` | All right + matching |
| `FULL JOIN`  | All records          |

## ⚡ 6. Window Function Quick Guide

| Function       | Use            |
| -------------- | -------------- |
| `ROW_NUMBER()` | Unique ranking |
| `RANK()`       | Skips ranks    |
| `DENSE_RANK()` | No gaps        |
| `LAG()`        | Previous row   |
| `LEAD()`       | Next row       |

## ⚡ 7. Indexing (Interview Shortcut)

**✔ Use index when:**

- Column used in `WHERE`
- JOIN condition
- `ORDER BY`

**❌ Avoid index when:**

- Small tables
- Frequent updates

## ⚡ 8. Keys Quick Recall

| Key           | Meaning           |
| ------------- | ----------------- |
| Primary Key   | Unique + NOT NULL |
| Foreign Key   | Reference         |
| Unique Key    | Unique values     |
| Composite Key | Multiple columns  |

## ⚡ 9. Common Mistakes (Interview Killers 🚫)

- Using `WHERE` instead of `HAVING`
- Missing JOIN condition → Cartesian product
- Not handling `NULL` properly
- Using subquery when `JOIN` is better
- Forgetting `GROUP BY` columns

## ⚡ 10. Problem Solving Approach (🔥 MUST FOLLOW)

When interviewer gives a question:

**Step 1: Understand**

What is required? (count, max, ranking)

**Step 2: Identify Pattern**

- Aggregation?
- Join?
- Window function?

**Step 3: Write Query**

- Start simple
- Then optimize

**Step 4: Optimize**

- Use index
- Reduce subqueries

## ⚡ 11. 1-Hour Revision Strategy

**⏱ First 20 min**

- Query templates
- Execution order
- JOIN types

**⏱ Next 20 min**

- Top patterns:
  - Second highest
  - Top N per group
  - Duplicates

**⏱ Last 20 min**

- Window functions
- Real questions

## 🧠 Final Interview Tip

If stuck:

👉 Say this:

“I’ll start with a basic approach, then optimize using window functions or indexing.”

This shows senior-level thinking.
