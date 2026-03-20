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
| INNER JOIN | Only matching records |
| LEFT JOIN | All from left + matches |
| RIGHT JOIN | All from right + matches |
| FULL JOIN | All records from both (via UNION in MySQL) |
| SELF JOIN | Table joined with itself |
| CROSS JOIN | Cartesian product |

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

## 🔥 Top 30 SQL JOIN Interview Questions

---

1️⃣ **INNER JOIN — Basic**

**Q:** Get employee names with department names.

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

2️⃣ **LEFT JOIN**

**Q:** Show all employees including those without departments.

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id;
```

3️⃣ **RIGHT JOIN**

**Q:** Show all departments even if no employees exist.

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.id;
```

4️⃣ **FULL JOIN (MySQL Trick)**

**Q:** Simulate FULL JOIN.

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

5️⃣ **Employees Without Department**

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id
WHERE d.id IS NULL;
```

6️⃣ **Departments Without Employees**

```sql
SELECT d.department_name
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
WHERE e.id IS NULL;
```

7️⃣ **JOIN with Filter**

```sql
SELECT e.name, d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.id
WHERE e.salary > 70000;
```

8️⃣ **Count Employees per Department**

```sql
SELECT d.department_name, COUNT(e.id)
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name;
```

9️⃣ **Highest Salary per Department**

```sql
SELECT d.department_name, MAX(e.salary)
FROM employees e
JOIN departments d
ON e.department_id = d.id
GROUP BY d.department_name;
```

🔟 **SELF JOIN (Very Important)**

**Q:** Find employee-manager relationship.

```sql
SELECT e.name AS employee,
       m.name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.id;
```

1️⃣1️⃣ **Customers Who Placed Orders**

```sql
SELECT c.customer_name
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id;
```

1️⃣2️⃣ **Customers Without Orders**

```sql
SELECT c.customer_name
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

1️⃣3️⃣ **Total Order Amount per Customer**

```sql
SELECT c.customer_name, SUM(o.amount)
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```

1️⃣4️⃣ **Highest Order per Customer**

```sql
SELECT c.customer_name, MAX(o.amount)
FROM customers c
JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_name;
```

1️⃣5️⃣ **Multi-Table JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

1️⃣6️⃣ **Find Duplicate Salaries (Self Join)**

```sql
SELECT a.id, b.id
FROM employees a
JOIN employees b
ON a.salary = b.salary
AND a.id <> b.id;
```

1️⃣7️⃣ **JOIN with Multiple Conditions**

```sql
SELECT *
FROM employees e
JOIN departments d
ON e.department_id = d.id
AND e.salary > 60000;
```

1️⃣8️⃣ **CROSS JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

1️⃣9️⃣ **USING Clause**

```sql
SELECT name, department_name
FROM employees
JOIN departments USING(department_id);
```

2️⃣0️⃣ **Employees in Engineering**

```sql
SELECT e.name
FROM employees e
JOIN departments d
ON e.department_id = d.id
WHERE d.department_name = 'Engineering';
```

2️⃣1️⃣ **Employees Earning Above Department Avg**

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary >
(
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

2️⃣2️⃣ **JOIN with Subquery**

```sql
SELECT name
FROM employees
WHERE department_id IN
(
    SELECT id FROM departments
    WHERE department_name = 'Engineering'
);
```

2️⃣3️⃣ **Highest Salary Employee**

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 1;
```

2️⃣4️⃣ **Second Highest Salary**

```sql
SELECT MAX(salary)
FROM employees
WHERE salary <
(
    SELECT MAX(salary)
    FROM employees
);
```

2️⃣5️⃣ **Aggregations with JOIN**

```sql
SELECT d.department_name,
       COUNT(e.id),
       AVG(e.salary),
       MAX(e.salary)
FROM departments d
LEFT JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name;
```

2️⃣6️⃣ **Employees in Same Department**

```sql
SELECT e1.name, e2.name
FROM employees e1
JOIN employees e2
ON e1.department_id = e2.department_id
AND e1.id <> e2.id;
```

2️⃣7️⃣ **Department with Highest Total Salary**

```sql
SELECT d.department_name, SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d
ON e.department_id = d.id
GROUP BY d.department_name
ORDER BY total_salary DESC
LIMIT 1;
```

2️⃣8️⃣ **Top 3 Highest Paid Employees**

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

2️⃣9️⃣ **Employees Without Orders**

```sql
SELECT e.name
FROM employees e
LEFT JOIN orders o
ON e.id = o.customer_id
WHERE o.order_id IS NULL;
```

3️⃣0️⃣ **Departments with More Than 2 Employees**

```sql
SELECT d.department_name
FROM departments d
JOIN employees e
ON d.id = e.department_id
GROUP BY d.department_name
HAVING COUNT(e.id) > 2;
```

---
# 🎯 Most Important Interview Patterns
---
## Focus heavily on these:

- INNER JOIN vs LEFT JOIN
- LEFT JOIN + NULL filtering
- SELF JOIN
- Aggregation with JOIN
- Multi-table JOIN
- Finding missing records
- Correlated subqueries
- Duplicate detection

## 🧪 Practice Tips

### Practice on:

- Employee-Department schema
- E-commerce schema

### Focus on:

- Writing queries without help
- Explaining JOIN logic clearly

### Optimize for:

- Readability
- Performance

---
