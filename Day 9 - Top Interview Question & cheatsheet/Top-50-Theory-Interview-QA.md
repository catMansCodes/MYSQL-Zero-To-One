✅ **Topic-wise Important Questions (Theory + Practical)**


## 📌 1. SQL Fundamentals (SELECT, WHERE, ORDER)

**Theoretical Questions**

- What is the difference between `WHERE` and `HAVING`?
- Execution order of SQL query?
- Difference between `DISTINCT` and `GROUP BY`?
- What is `LIMIT` and how is it used?
- Difference between `=` and `IN`?

**Practical Questions**

Q1: Find employees with salary > 50,000

```sql
SELECT *
FROM employees
WHERE salary > 50000;
```

Q2: Get top 3 highest salaries

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

Q3: Find employees whose name starts with 'A'

```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

## 📌 2. Aggregations & GROUP BY

**Theoretical Questions**

- Difference between `COUNT(*)` and `COUNT(column)`?
- Can we use `WHERE` with aggregate functions?
- Difference between `WHERE` and `HAVING`?
- What happens if we don’t use `GROUP BY` with aggregate?

**Practical Questions**

Q1: Find total salary per department

```sql
SELECT department, SUM(salary)
FROM employees
GROUP BY department;
```

Q2: Departments with avg salary > 60k

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;
```

Q3: Count employees in each department

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

## 📌 3. JOINs (🔥 MOST IMPORTANT)

**Theoretical Questions**

- Difference between `INNER JOIN` and `LEFT JOIN`?
- What is `SELF JOIN`?
- Difference between `JOIN` and `UNION`?
- What happens if join condition is missing?

**Practical Questions**

Q1: Get employee name with department name

```sql
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d
ON e.dept_id = d.dept_id;
```

Q2: Find employees without department

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d
ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

Q3: Self join (manager hierarchy)

```sql
SELECT e.name, m.name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.emp_id;
```

## 📌 4. Subqueries

**Theoretical Questions**

- What is correlated vs non-correlated subquery?
- Difference between `IN` and `EXISTS`?
- Which is faster: `JOIN` or subquery?

**Practical Questions**

Q1: Employees earning more than average salary

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) FROM employees
);
```

Q2: Find employees in departments with >5 employees

```sql
SELECT name
FROM employees
WHERE dept_id IN (
    SELECT dept_id
    FROM employees
    GROUP BY dept_id
    HAVING COUNT(*) > 5
);
```

## 📌 5. Window Functions (🔥 Advanced + High Weightage)

**Theoretical Questions**

- Difference between `RANK()` and `ROW_NUMBER()`?
- What is `PARTITION BY`?
- When to use window functions vs `GROUP BY`?

**Practical Questions**

Q1: Rank employees by salary

```sql
SELECT name, salary,
RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

Q2: Top 2 salaries per department

```sql
SELECT *
FROM (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) rn
    FROM employees
) t
WHERE rn <= 2;
```

## 📌 6. Indexing & Performance

**Theoretical Questions**

- What is an index?
- Difference between clustered and non-clustered index?
- When should you avoid indexes?
- What is query optimization?

**Practical Questions**

Q1: Create index on salary

```sql
CREATE INDEX idx_salary
ON employees(salary);
```

Q2: Check query performance

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

## 📌 7. Keys & Constraints (Important for Design)

**Theoretical Questions**

- Difference between Primary Key and Foreign Key?
- What is composite key?
- Can foreign key reference unique key?
- What is cascading?

**Practical Questions**

Q1: Create table with constraints

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Q2: Cascade delete

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE;
```

## 📌 8. Database Design (System Design Level SQL)

**Theoretical Questions**

- What is normalization?
- Difference between 1NF, 2NF, 3NF?
- When to denormalize?
- What is many-to-many relationship?

**Practical Questions**

Q1: Design many-to-many relation

```sql
CREATE TABLE student_course (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id)
);
```
