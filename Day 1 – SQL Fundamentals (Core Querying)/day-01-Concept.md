# 📘 Day 1 — SQL Fundamentals (Core Querying)

## 🎯 Objective

Learn the foundational SQL querying concepts required for backend development and technical interviews.

---

# 📂 Dataset Setup

## 🏗️ Create Table

```sql
CREATE TABLE employees (
    id INT,
    name VARCHAR(50),
    department VARCHAR(50),
    salary INT,
    city VARCHAR(50)
);
```

## 📥 Insert Sample Data

```sql
INSERT INTO employees VALUES
(1,'Rahul','Engineering',90000,'Delhi'),
(2,'Amit','HR',60000,'Mumbai'),
(3,'Neha','Engineering',95000,'Bangalore'),
(4,'Priya','Finance',70000,'Delhi'),
(5,'Karan','Engineering',85000,'Mumbai'),
(6,'Anita','HR',65000,'Delhi');
```

---

# 📚 Core Concepts

## 1️⃣ SELECT

Retrieve data from table

```sql
SELECT * FROM employees;

SELECT name, salary FROM employees;
```

⚠️ Avoid `SELECT *` in production.

---

## 2️⃣ WHERE

Filter rows

```sql
SELECT * FROM employees
WHERE department = 'Engineering';
```

---

## 3️⃣ Operators

| Operator | Example            |
| -------- | ------------------ |
| =        | department = 'HR'  |
| != / <>  | department != 'HR' |
| >        | salary > 80000     |
| <        | salary < 70000     |
| >=       | salary >= 90000    |

---

## 4️⃣ AND / OR

```sql
SELECT * FROM employees
WHERE department = 'Engineering'
AND salary > 90000;

SELECT * FROM employees
WHERE city = 'Delhi'
OR city = 'Mumbai';
```

---

## 5️⃣ ORDER BY

```sql
SELECT * FROM employees
ORDER BY salary DESC;

SELECT * FROM employees
ORDER BY department, salary DESC;
```

---

## 6️⃣ LIMIT

```sql
SELECT * FROM employees
LIMIT 3;

SELECT * FROM employees
LIMIT 10 OFFSET 20;
```

---

## 7️⃣ DISTINCT

```sql
SELECT DISTINCT department
FROM employees;
```

---

## 8️⃣ IN

```sql
SELECT * FROM employees
WHERE city IN ('Delhi','Mumbai');
```

---

## 9️⃣ BETWEEN

```sql
SELECT * FROM employees
WHERE salary BETWEEN 70000 AND 90000;
```

---

## 🔟 LIKE

```sql
-- Starts with A
SELECT * FROM employees WHERE name LIKE 'A%';

-- Ends with a
SELECT * FROM employees WHERE name LIKE '%a';

-- Contains h
SELECT * FROM employees WHERE name LIKE '%h%';
```

---

# ⚙️ SQL Execution Order (Important Interview Question)

```text
FROM → WHERE → SELECT → ORDER BY → LIMIT
```

---

# 💼 Interview Questions & Answers

## 1. Get employees from Delhi

```sql
SELECT * FROM employees
WHERE city = 'Delhi';
```

---

## 2. Salary > 80000

```sql
SELECT * FROM employees
WHERE salary > 80000;
```

---

## 3. Unique departments

```sql
SELECT DISTINCT department
FROM employees;
```

---

## 4. Employees from Delhi or Mumbai

```sql
SELECT * FROM employees
WHERE city IN ('Delhi','Mumbai');
```

---

## 5. Sort by salary (highest first)

```sql
SELECT * FROM employees
ORDER BY salary DESC;
```

---

## 6. Top 3 highest paid employees

```sql
SELECT * FROM employees
ORDER BY salary DESC
LIMIT 3;
```

---

## 7. Names starting with A

```sql
SELECT * FROM employees
WHERE name LIKE 'A%';
```

---

## 8. Names ending with a

```sql
SELECT * FROM employees
WHERE name LIKE '%a';
```

---

## 9. Salary between 70000 and 90000

```sql
SELECT * FROM employees
WHERE salary BETWEEN 70000 AND 90000;
```

---

## 10. Engineering + salary > 85000

```sql
SELECT * FROM employees
WHERE department = 'Engineering'
AND salary > 85000;
```

---

## 11. Select only name & salary

```sql
SELECT name, salary FROM employees;
```

---

## 12. NOT HR employees

```sql
SELECT * FROM employees
WHERE department != 'HR';
```

---

## 13. City starts with B

```sql
SELECT * FROM employees
WHERE city LIKE 'B%';
```

---

## 14. Sort by department & salary

```sql
SELECT * FROM employees
ORDER BY department, salary DESC;
```

---

## 15. Salary NOT between 60000 and 80000

```sql
SELECT * FROM employees
WHERE salary NOT BETWEEN 60000 AND 80000;
```

---

# 🧠 Practice Questions

Try solving without looking:

1. Employees from Engineering OR HR
2. Names containing 'a'
3. Delhi employees with salary > 70000
4. Top 2 highest paid employees
5. Second highest salary

---

# 🚀 Key Takeaways

* Master **SELECT + WHERE + ORDER BY**
* Learn filtering using **IN, BETWEEN, LIKE**
* Avoid `SELECT *` in real-world applications
* Understand query execution order (very common interview question)

---
# 📌 Tip for Interviews

> Most SQL questions are just combinations of:
>
> * Filtering (WHERE)
> * Sorting (ORDER BY)
> * Limiting (LIMIT)

