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

```
-- Employees Table
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT
);

-- Orders Table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    amount DECIMAL(10,2),
    order_date DATE
);

-- Insert records employee
INSERT INTO employees (emp_id, name, department, salary, hire_date, manager_id) VALUES
(1, 'John', 'Engineering', 90000, '2020-01-10', 101),
(2, 'Alice', 'Engineering', 85000, '2021-03-15', 101),
(3, 'Bob', 'HR', 60000, '2019-07-12', 102),
(4, 'Eva', 'HR', 65000, '2022-04-21', 102),
(5, 'David', 'Sales', 70000, '2021-05-01', 103),
(6, 'Sam', 'Sales', 72000, '2022-08-19', 103),
(7, 'Kevin', 'Engineering', 95000, '2018-06-25', 101),
(8, 'Tom', 'Sales', 68000, '2020-09-14', 103);

-- Insert record order
INSERT INTO orders (order_id, customer_id, amount, order_date) VALUES
(1, 201, 500, '2023-01-01'),
(2, 202, 700, '2023-01-02'),
(3, 201, 300, '2023-01-05'),
(4, 203, 900, '2023-01-07'),
(5, 204, 1200, '2023-01-10'),
(6, 202, 400, '2023-01-12');

```

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



# 🧪 Practice Questions (Must Do)

1. Highest Revenue Customer
SELECT customer_id,
       SUM(amount) AS total_revenue
FROM orders
GROUP BY customer_id
ORDER BY total_revenue DESC
LIMIT 1;
Output
customer_id	total_revenue
204	1200
2. Average Order Value
SELECT AVG(amount) AS avg_order_value
FROM orders;
Output
666.67
3. Team Size Per Manager
SELECT manager_id,
       COUNT(*) AS team_size
FROM employees
GROUP BY manager_id;
Output
manager_id	team_size
101	3
102	2
103	3
4. Sales Per Day
SELECT order_date,
       SUM(amount) AS total_sales
FROM orders
GROUP BY order_date
ORDER BY order_date;
Output
order_date	total_sales
2023-01-01	500
2023-01-02	700
2023-01-05	300
2023-01-07	900
2023-01-10	1200
2023-01-12	400
5. Departments Above Company Average Salary
Company Average Salary
SELECT AVG(salary)
FROM employees;

Average = 75625

Departments Above Average
SELECT department,
       AVG(salary) AS dept_avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) >
(
    SELECT AVG(salary)
    FROM employees
);
Output
department	dept_avg_salary
Engineering	90000
6. Top 3 Highest Paid Employees Per Department
Window Function Solution (Interview Favorite)
SELECT *
FROM
(
    SELECT emp_id,
           name,
           department,
           salary,
           DENSE_RANK() OVER
           (
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rnk
    FROM employees
) t
WHERE rnk <= 3;
Output

Engineering

Name	Salary
Kevin	95000
John	90000
Alice	85000

HR

Name	Salary
Eva	65000
Bob	60000

Sales

Name	Salary
Sam	72000
David	70000
Tom	68000
7. Department With Lowest Average Salary
SELECT department,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary
LIMIT 1;
Output
department	avg_salary
HR	62500
8. Customer With Maximum Orders
SELECT customer_id,
       COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id
ORDER BY total_orders DESC
LIMIT 1;
Output
customer_id	total_orders
201	2

Customer 202 also has 2 orders, so there is a tie.

Tie-Safe Solution
SELECT customer_id,
       COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id
HAVING COUNT(*) =
(
    SELECT MAX(order_count)
    FROM
    (
        SELECT COUNT(*) AS order_count
        FROM orders
        GROUP BY customer_id
    ) t
);
9. Date With Highest Revenue
SELECT order_date,
       SUM(amount) AS revenue
FROM orders
GROUP BY order_date
ORDER BY revenue DESC
LIMIT 1;
Output
order_date	revenue
2023-01-10	1200
10. Employees Hired Per Month
MySQL
SELECT DATE_FORMAT(hire_date,'%Y-%m') AS month,
       COUNT(*) AS employees_hired
FROM employees
GROUP BY DATE_FORMAT(hire_date,'%Y-%m')
ORDER BY month;
Output
month	employees_hired
2018-06	1
2019-07	1
2020-01	1
2020-09	1
2021-03	1
2021-05	1
2022-04	1
2022-08	1

