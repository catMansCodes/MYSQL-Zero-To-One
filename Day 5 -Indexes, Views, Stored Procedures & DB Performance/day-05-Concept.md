# 📘 Day 5 — Indexes, Views, Stored Procedures & Performance (SQL Interview Guide)

This day focuses on **advanced database concepts** that are heavily asked in **backend & system design interviews**.

You’ll learn:

* Indexing (Performance Optimization)
* Views (Abstraction Layer)
* Stored Procedures (Business Logic in DB)
* Triggers (Automation)
* Transactions (ACID Properties)
* Constraints
* Query Optimization Techniques

---

# 🧪 Practice Dataset

## Employees Table

| id | name    | department_id | salary | hire_date  |
| -- | ------- | ------------- | ------ | ---------- |
| 1  | Alice   | 1             | 80000  | 2021-02-01 |
| 2  | Bob     | 2             | 60000  | 2022-01-10 |
| 3  | Charlie | 1             | 90000  | 2020-05-12 |
| 4  | David   | 3             | 70000  | 2023-04-01 |
| 5  | Eva     | 2             | 75000  | 2021-07-15 |

## Departments Table

| id | department_name |
| -- | --------------- |
| 1  | Engineering     |
| 2  | HR              |
| 3  | Finance         |

---

# 🚀 1. Indexing (Most Important for Performance)

## 🔹 What is an Index?

An index improves query performance by avoiding full table scans.

---

## 🔹 Create Index

```sql
CREATE INDEX idx_employee_salary
ON employees(salary);
```

---

## 🔹 Composite Index

```sql
CREATE INDEX idx_dept_salary
ON employees(department_id, salary);
```

---

## 🔹 Example Query

```sql
SELECT *
FROM employees
WHERE salary > 70000;
```

---

## 🔹 Check Execution Plan

```sql
EXPLAIN SELECT *
FROM employees
WHERE salary > 70000;
```

---

## 🔹 Important Notes

* Too many indexes slow down INSERT/UPDATE
* Order matters in the composite index

---

# 🧩 2. Views

## 🔹 What is a View?

A virtual table created using a query.

---

## 🔹 Create View

```sql
CREATE VIEW employee_department_view AS
SELECT e.name, d.department_name, e.salary
FROM employees e
JOIN departments d
ON e.department_id = d.id;
```

---

## 🔹 Use View

```sql
SELECT * FROM employee_department_view;
```

---

## 🔹 Advantages

* Simplifies complex queries
* Improves security
* Reusable logic

---

# ⚙️ 3. Stored Procedures

## 🔹 What is a Stored Procedure?

Precompiled SQL logic stored in the database.

---

## 🔹 Example

```sql
DELIMITER //

CREATE PROCEDURE increase_salary(
    IN dept_id INT,
    IN increment_amount INT
)
BEGIN
    UPDATE employees
    SET salary = salary + increment_amount
    WHERE department_id = dept_id;
END //

DELIMITER ;
```

---

## 🔹 Execute

```sql
CALL increase_salary(1, 5000);
```

---

# 🔄 4. Triggers

## 🔹 What is Trigger?

Automatically executes on INSERT/UPDATE/DELETE.

---

## 🔹 Audit Table

```sql
CREATE TABLE salary_audit (
    emp_id INT,
    old_salary INT,
    new_salary INT
);
```

---

## 🔹 Trigger Example

```sql
CREATE TRIGGER salary_update_trigger
AFTER UPDATE ON employees
FOR EACH ROW
INSERT INTO salary_audit
VALUES (OLD.id, OLD.salary, NEW.salary);
```

---

# 🔐 5. Transactions (ACID Properties)

## 🔹 Transaction Example

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

---

## 🔹 Rollback

```sql
ROLLBACK;
```

---

## 🔹 ACID

| Property    | Meaning         |
| ----------- | --------------- |
| Atomicity   | All or nothing  |
| Consistency | Valid state     |
| Isolation   | No interference |
| Durability  | Permanent data  |

---

# 📌 6. Constraints

## 🔹 Primary Key

```sql
id INT PRIMARY KEY
```

## 🔹 Foreign Key

```sql
FOREIGN KEY (department_id)
REFERENCES departments(id)
```

## 🔹 Unique

```sql
email VARCHAR(100) UNIQUE
```

## 🔹 Not Null

```sql
name VARCHAR(50) NOT NULL
```

---

# ⚡ 7. Query Optimization

## ❌ Bad Query

```sql
SELECT *
FROM employees
WHERE YEAR(hire_date) = 2021;
```

---

## ✅ Optimized Query

```sql
SELECT *
FROM employees
WHERE hire_date BETWEEN '2021-01-01'
AND '2021-12-31';
```

---

# 🎯 Top 10 Interview Questions (Day 5)

## Q1: Clustered vs Non-Clustered Index

Clustered → data stored physically
Non-clustered → separate structure

---

## Q2: What happens with too many indexes?

* Slow writes
* More storage
* Maintenance overhead

---

## Q3: View vs Materialized View

View → virtual
Materialized → stored

---

## Q4: Stored Procedure vs Function

| Stored Procedure | Function       |
| ---------------- | -------------- |
| Multiple outputs | Single value   |
| Can modify data  | Used in SELECT |

---

## Q5: Trigger vs Stored Procedure

Trigger → automatic
Procedure → manual

---

## Q6: What is an Execution Plan?

```sql
EXPLAIN SELECT ...
```

---

## Q7: What is Normalization?

Removing redundancy

---

## Q8: What is Denormalization?

Adding redundancy for performance

---

## Q9: What is Deadlock?

Two transactions are waiting on each other

---

## Q10: Isolation Levels

* READ UNCOMMITTED
* READ COMMITTED
* REPEATABLE READ
* SERIALIZABLE

---
