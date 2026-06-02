# Day 6 — Top 50 Real SQL Interview Questions

*Amazon, Flipkart, Google Style*

---

## Dataset Setup

```sql
CREATE TABLE customers (
    id   INT PRIMARY KEY,
    name VARCHAR(50),
    city VARCHAR(50)
);

CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT,
    order_date  DATE,
    amount      DECIMAL(10,2)
);

CREATE TABLE products (
    id       INT PRIMARY KEY,
    name     VARCHAR(50),
    category VARCHAR(50),
    price    DECIMAL(10,2)
);

CREATE TABLE order_items (
    order_id   INT,
    product_id INT,
    quantity   INT
);

CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(50),
    department VARCHAR(50),
    salary     INT,
    manager_id INT
);
```

```sql
INSERT INTO customers VALUES
(1,'Alice','Delhi'),(2,'Bob','Mumbai'),(3,'Charlie','Bangalore'),
(4,'Diana','Delhi'),(5,'Eve','Mumbai'),(6,'Frank','Pune');

INSERT INTO orders VALUES
(1,1,'2024-01-05',1500.00),(2,1,'2024-01-06',2300.00),
(3,2,'2024-02-15',800.00),(4,3,'2024-02-01',4200.00),
(5,1,'2024-02-10',900.00),(6,2,'2024-02-11',1100.00),
(7,3,'2024-03-01',600.00),(8,1,'2024-03-05',3200.00),
(9,4,'2024-03-10',2500.00),(10,5,'2024-03-15',700.00);

INSERT INTO products VALUES
(1,'Laptop','Electronics',80000.00),(2,'Phone','Electronics',50000.00),
(3,'Shirt','Clothing',1200.00),(4,'Pants','Clothing',2000.00),
(5,'Book','Education',500.00),(6,'Tablet','Electronics',30000.00);

INSERT INTO order_items VALUES
(1,1,1),(1,3,2),(2,2,1),(3,4,3),(4,1,1),
(4,2,2),(5,5,5),(6,3,1),(7,6,1),(8,1,1),(9,2,1),(10,4,2);

INSERT INTO employees VALUES
(1,'Rahul','Engineering',90000,5),(2,'Amit','HR',60000,6),
(3,'Neha','Engineering',95000,5),(4,'Priya','Finance',70000,6),
(5,'Karan','Engineering',110000,NULL),(6,'Anita','HR',80000,NULL),
(7,'Ravi','Engineering',85000,5),(8,'Sonia','Finance',65000,6);
```

---

---

# 1. Basic SQL (Q1–Q10)

### Q1. Customers who placed at least one order

```sql
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

---

### Q2. Total number of orders

```sql
SELECT COUNT(*) AS total_orders FROM orders;
```

---

### Q3. Total revenue

```sql
SELECT SUM(amount) AS total_revenue FROM orders;
```

---

### Q4. Customers with no orders

```sql
SELECT name
FROM customers
WHERE id NOT IN (SELECT customer_id FROM orders);
```

---

### Q5. Highest single order amount

```sql
SELECT MAX(amount) AS max_order FROM orders;
```

---

### Q6. Top 5 most expensive products

```sql
SELECT name, price
FROM products
ORDER BY price DESC
LIMIT 5;
```

---

### Q7. Number of orders per customer

```sql
SELECT customer_id, COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id;
```

---

### Q8. Average product price by category

```sql
SELECT category, ROUND(AVG(price), 2) AS avg_price
FROM products
GROUP BY category;
```

---

### Q9. Employees earning above company average

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

### Q10. Second highest salary

```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

---

# 2. Intermediate SQL (Q11–Q25)

### Q11. Customers with more than 2 orders

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 2;
```

---

### Q12. Most ordered product (by quantity)

```sql
SELECT product_id, SUM(quantity) AS total_qty
FROM order_items
GROUP BY product_id
ORDER BY total_qty DESC
LIMIT 1;
```

---

### Q13. Total revenue per customer

```sql
SELECT c.name, SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

---

### Q14. Latest order date per customer

```sql
SELECT customer_id, MAX(order_date) AS last_order
FROM orders
GROUP BY customer_id;
```

---

### Q15. Customers who ordered on at least 3 distinct dates

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS active_days
FROM orders
GROUP BY customer_id
HAVING COUNT(DISTINCT order_date) >= 3;
```

---

### Q16. Duplicate customer names

```sql
SELECT name, COUNT(*) AS occurrences
FROM customers
GROUP BY name
HAVING COUNT(*) > 1;
```

---

### Q17. Employees with the same salary

```sql
SELECT salary, COUNT(*) AS count
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

---

### Q18. Total quantity of items per order

```sql
SELECT order_id, SUM(quantity) AS total_items
FROM order_items
GROUP BY order_id;
```

---

### Q19. Customers who spent more than 5000 total

```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 5000;
```

---

### Q20. Products that were never ordered

```sql
SELECT name
FROM products
WHERE id NOT IN (SELECT DISTINCT product_id FROM order_items);
```

---

### Q21. First order placed (globally)

```sql
SELECT * FROM orders ORDER BY order_date LIMIT 1;
```

---

### Q22. Number of orders per city

```sql
SELECT c.city, COUNT(o.id) AS order_count
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.city;
```

---

### Q23. Top customer by total revenue

```sql
SELECT c.name, SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name
ORDER BY total_spent DESC
LIMIT 1;
```

---

### Q24. Products ordered more than 3 times (quantity total)

```sql
SELECT product_id, SUM(quantity) AS total_qty
FROM order_items
GROUP BY product_id
HAVING SUM(quantity) > 3;
```

---

### Q25. Department with the highest average salary

```sql
SELECT department, ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

---

---

# 3. Advanced SQL (Q26–Q40)

### Q26. 3rd highest salary

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

---

### Q27. Rank all employees by salary

```sql
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) AS rnk
FROM employees;
```

---

### Q28. Top 3 salaries per department

```sql
SELECT name, department, salary
FROM (
    SELECT name, department, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk <= 3;
```

---

### Q29. Running total of order amounts (by date)

```sql
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

---

### Q30. Previous order amount per customer

```sql
SELECT customer_id, order_date, amount,
       LAG(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_amount
FROM orders;
```

---

### Q31. Daily revenue and day-over-day growth

```sql
WITH daily AS (
    SELECT order_date, SUM(amount) AS daily_revenue
    FROM orders
    GROUP BY order_date
)
SELECT order_date,
       daily_revenue,
       LAG(daily_revenue) OVER (ORDER BY order_date) AS prev_day_revenue,
       daily_revenue - LAG(daily_revenue) OVER (ORDER BY order_date) AS growth
FROM daily;
```

---

### Q32. First order above 1000 per customer

```sql
SELECT customer_id, order_date, amount
FROM (
    SELECT customer_id, order_date, amount,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rn
    FROM orders
    WHERE amount > 1000
) t
WHERE rn = 1;
```

---

### Q33. Each product's revenue contribution %

```sql
SELECT p.name,
       SUM(oi.quantity * p.price) AS product_revenue,
       ROUND(100.0 * SUM(oi.quantity * p.price) /
             SUM(SUM(oi.quantity * p.price)) OVER (), 2) AS revenue_pct
FROM order_items oi
JOIN products p ON oi.product_id = p.id
GROUP BY p.name;
```

---

### Q34. Top selling category (by quantity)

```sql
SELECT p.category, SUM(oi.quantity) AS total_qty
FROM order_items oi
JOIN products p ON oi.product_id = p.id
GROUP BY p.category
ORDER BY total_qty DESC
LIMIT 1;
```

---

### Q35. Churned customers (no order in last 90 days)

```sql
SELECT c.name, MAX(o.order_date) AS last_order_date
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name
HAVING MAX(o.order_date) < CURDATE() - INTERVAL 90 DAY;
```

---

### Q36. Products frequently bought together (same order)

```sql
SELECT a.product_id AS product1,
       b.product_id AS product2,
       COUNT(*) AS times_together
FROM order_items a
JOIN order_items b ON a.order_id = b.order_id
                  AND a.product_id < b.product_id
GROUP BY a.product_id, b.product_id
ORDER BY times_together DESC;
```

---

### Q37. Longest gap between orders per customer

```sql
WITH gaps AS (
    SELECT customer_id, order_date,
           LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_date
    FROM orders
)
SELECT customer_id,
       MAX(DATEDIFF(order_date, prev_date)) AS max_gap_days
FROM gaps
WHERE prev_date IS NOT NULL
GROUP BY customer_id;
```

---

### Q38. Employees earning more than their manager

```sql
SELECT e.name AS employee, e.salary,
       m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### Q39. Median salary

```sql
SELECT AVG(salary) AS median_salary
FROM (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER ()                    AS total
    FROM employees
) t
WHERE rn IN (FLOOR((total + 1) / 2), CEIL((total + 1) / 2));
```

---

### Q40. Monthly revenue with cumulative total

```sql
WITH monthly AS (
    SELECT DATE_FORMAT(order_date, '%Y-%m') AS month,
           SUM(amount) AS monthly_revenue
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT month, monthly_revenue,
       SUM(monthly_revenue) OVER (ORDER BY month) AS cumulative_revenue
FROM monthly;
```

---

---

# 4. Hard / FAANG Level (Q41–Q50)

### Q41. Customers who ordered on consecutive days

```sql
WITH ordered AS (
    SELECT customer_id, order_date,
           LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_date
    FROM orders
)
SELECT DISTINCT customer_id
FROM ordered
WHERE DATEDIFF(order_date, prev_date) = 1;
```

---

### Q42. Most popular product per city

```sql
SELECT city, product_id, total_qty
FROM (
    SELECT c.city, oi.product_id,
           SUM(oi.quantity) AS total_qty,
           RANK() OVER (PARTITION BY c.city ORDER BY SUM(oi.quantity) DESC) AS rnk
    FROM customers c
    JOIN orders o    ON c.id = o.customer_id
    JOIN order_items oi ON o.id = oi.order_id
    GROUP BY c.city, oi.product_id
) t
WHERE rnk = 1;
```

---

### Q43. Suspicious customers — multiple orders on the same day

```sql
SELECT customer_id, order_date, COUNT(*) AS orders_on_day
FROM orders
GROUP BY customer_id, order_date
HAVING COUNT(*) > 1;
```

---

### Q44. Order sessions — group orders within 7 days of each other per customer

```sql
WITH gaps AS (
    SELECT customer_id, order_date,
           CASE WHEN DATEDIFF(order_date,
                    LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)) > 7
                THEN 1 ELSE 0 END AS new_session
    FROM orders
),
sessions AS (
    SELECT customer_id, order_date,
           SUM(new_session) OVER (PARTITION BY customer_id ORDER BY order_date) AS session_id
    FROM gaps
)
SELECT customer_id, session_id,
       MIN(order_date) AS session_start,
       MAX(order_date) AS session_end,
       COUNT(*) AS orders_in_session
FROM sessions
GROUP BY customer_id, session_id;
```

---

### Q45. Active vs churned customers (last 60 days threshold)

```sql
SELECT c.name,
       MAX(o.order_date) AS last_order,
       CASE WHEN MAX(o.order_date) >= CURDATE() - INTERVAL 60 DAY
            THEN 'Active'
            ELSE 'Churned'
       END AS status
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

---

### Q46. Top product per category (by quantity sold)

```sql
SELECT category, name, total_qty
FROM (
    SELECT p.category, p.name,
           SUM(oi.quantity) AS total_qty,
           RANK() OVER (PARTITION BY p.category ORDER BY SUM(oi.quantity) DESC) AS rnk
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.category, p.name
) t
WHERE rnk = 1;
```

---

### Q47. Revenue share % per category

```sql
SELECT p.category,
       SUM(oi.quantity * p.price) AS category_revenue,
       ROUND(100.0 * SUM(oi.quantity * p.price) /
             SUM(SUM(oi.quantity * p.price)) OVER (), 2) AS revenue_pct
FROM order_items oi
JOIN products p ON oi.product_id = p.id
GROUP BY p.category;
```

---

### Q48. Customers who bought every product in the Electronics category

```sql
SELECT o.customer_id
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p     ON oi.product_id = p.id
WHERE p.category = 'Electronics'
GROUP BY o.customer_id
HAVING COUNT(DISTINCT p.id) = (
    SELECT COUNT(*) FROM products WHERE category = 'Electronics'
);
```

---

### Q49. Second purchase date per customer

```sql
SELECT customer_id, order_date AS second_order_date
FROM (
    SELECT customer_id, order_date,
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rn
    FROM orders
) t
WHERE rn = 2;
```

---

### Q50. Most profitable customer cohort (grouped by first order month)

```sql
WITH cohort AS (
    SELECT customer_id,
           DATE_FORMAT(MIN(order_date), '%Y-%m') AS cohort_month
    FROM orders
    GROUP BY customer_id
)
SELECT c.cohort_month,
       COUNT(DISTINCT o.customer_id) AS customers,
       SUM(o.amount)                 AS total_revenue
FROM cohort c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.cohort_month
ORDER BY total_revenue DESC;
```

---

## Key Patterns to Remember

| Pattern | Technique |
|---------|-----------|
| Customers with no X | `LEFT JOIN ... WHERE right.id IS NULL` or `NOT IN` |
| Top N per group | `DENSE_RANK() OVER (PARTITION BY ... ORDER BY ...)` |
| Running total | `SUM() OVER (ORDER BY ...)` |
| Day-over-day change | `LAG()` in a CTE |
| % of total | `col / SUM(col) OVER () * 100` |
| Bought everything in category | `COUNT(DISTINCT) = (SELECT COUNT(*) ...)` |
| Consecutive events | `LAG(date)` + `DATEDIFF = 1` |
| Cohort analysis | `MIN(order_date)` grouped by customer → join back |
