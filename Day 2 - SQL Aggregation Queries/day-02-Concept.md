# 📘 Day 2 — SQL Aggregation Queries (Complete Guide)

---

## 🎯 Objective

Master **SQL Aggregation Functions + GROUP BY + HAVING** and solve **Top 20 Interview Questions** based on real-world datasets.

---

# 📊 Dataset Used (Interview Standard)

## 🧑‍💼 employees

| emp_id | name  | department  | salary | hire_date  | manager_id |
| ------ | ----- | ----------- | ------ | ---------- | ---------- |
| 1      | John  | Engineering | 90000  | 2020-01-10 | 101        |
| 2      | Alice | Engineering | 85000  | 2021-03-15 | 101        |
| 3      | Bob   | HR          | 60000  | 2019-07-12 | 102        |
| 4      | Eva   | HR          | 65000  | 2022-04-21 | 102        |
| 5      | David | Sales       | 70000  | 2021-05-01 | 103        |
| 6      | Sam   | Sales       | 72000  | 2022-08-19 | 103        |
| 7      | Kevin | Engineering | 95000  | 2018-06-25 | 101        |
| 8      | Tom   | Sales       | 68000  | 2020-09-14 | 103        |

---

## 🛒 orders

| order_id | customer_id | amount | order_date |
| -------- | ----------- | ------ | ---------- |
| 1        | 201         | 500    | 2023-01-01 |
| 2        | 202         | 700    | 2023-01-02 |
| 3        | 201         | 300    | 2023-01-05 |
| 4        | 203         | 900    | 2023-01-07 |
| 5        | 204         | 1200   | 2023-01-10 |
| 6        | 202         | 400    | 2023-01-12 |

---

# 🔑 Core Concepts

## 1. Aggregation Functions

| Function | Description |
| -------- | ----------- |
| COUNT()  | Count rows  |
| SUM()    | Total value |
| AVG()    | Average     |
| MIN()    | Minimum     |
| MAX()    | Maximum     |

---

## 2. GROUP BY

Group rows with same values.

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

---

## 3. HAVING vs WHERE

| WHERE        | HAVING               |
| ------------ | -------------------- |
| Filters rows | Filters grouped data |

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

---

## 4. Execution Order

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY

---

# 🧠 Top 20 Interview Questions (With Answers)

---

## 🟢 Easy Level

### 1. Total employees

```sql
SELECT COUNT(*) FROM employees;
```

---

### 2. Average salary

```sql
SELECT AVG(salary) FROM employees;
```

---

### 3. Highest salary

```sql
SELECT MAX(salary) FROM employees;
```

---

### 4. Lowest salary

```sql
SELECT MIN(salary) FROM employees;
```

---

### 5. Total salary expense

```sql
SELECT SUM(salary) FROM employees;
```

---

### 6. Employees per department

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

---

## 🟡 Medium Level

### 7. Average salary per department

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

---

### 8. Departments with more than 2 employees

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

---

### 9. Total salary per department

```sql
SELECT department, SUM(salary)
FROM employees
GROUP BY department;
```

---

### 10. Department with highest avg salary

```sql
SELECT department, AVG(salary) avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

---

### 11. Orders per customer

```sql
SELECT customer_id, COUNT(*)
FROM orders
GROUP BY customer_id;
```

---

### 12. Total sales

```sql
SELECT SUM(amount) FROM orders;
```

---

### 13. Average order value

```sql
SELECT AVG(amount) FROM orders;
```

---

### 14. Revenue per day

```sql
SELECT order_date, SUM(amount)
FROM orders
GROUP BY order_date;
```

---

## 🔴 Advanced Level

### 15. Customer who spent the most

```sql
SELECT customer_id, SUM(amount) total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 1;
```

---

### 16. Duplicate customers

```sql
SELECT customer_id, COUNT(*)
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

---

### 17. Manager with largest team

```sql
SELECT manager_id, COUNT(*) team_size
FROM employees
GROUP BY manager_id
ORDER BY team_size DESC
LIMIT 1;
```

---

### 18. Departments with salary > 200000

```sql
SELECT department, SUM(salary)
FROM employees
GROUP BY department
HAVING SUM(salary) > 200000;
```

---

### 19. Second highest salary

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

### 20. Employees earning above dept average

```sql
SELECT *
FROM employees e
WHERE salary >
(
    SELECT AVG(salary)
    FROM employees
    WHERE department = e.department
);
```

---

# ⚠️ Important Interview Concepts

## COUNT(*) vs COUNT(column)

```sql
COUNT(*)       -- includes NULL
COUNT(column)  -- ignores NULL
```

---

## Nested Aggregation

```sql
SELECT department
FROM employees
GROUP BY department
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

---

## Index Optimization Tip

For performance:

```sql
CREATE INDEX idx_customer ON orders(customer_id);
```

---

# 💼 Real Interview Questions (Asked in Product Companies)

1. Highest revenue customer
2. Avg order value
3. Team size per manager
4. Sales per day
5. Departments above company avg

---

# 🧪 Practice Questions (Must Do)

1. Top 3 highest paid employees per department
2. Department with lowest avg salary
3. Customer with maximum orders
4. Date with highest revenue
5. Employees hired per month

---

# 🚀 Summary

Today you learned:

* Aggregation functions
* GROUP BY & HAVING
* Real-world analytics queries
* Interview-level SQL patterns

---

# 📅 Next Step

👉 **Day 3 — SQL JOINs (Most Important Topic)**

Will cover:

* INNER / LEFT / RIGHT / FULL JOIN
* Real interview problems
* 30+ questions

---

💡 Tip: Aggregation + JOIN = 80% of SQL interview questions.
