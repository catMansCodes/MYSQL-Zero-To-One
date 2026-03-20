# 📅 Day 6 — Top 50 Real SQL Interview Questions

*(Amazon, Flipkart, Google Style)*

---

## 🎯 Goal

Master **real-world SQL interview questions** asked in top product-based companies like Amazon, Flipkart, and Google.

This day focuses on:

* Real business problems
* Analytical SQL thinking
* Window functions
* Complex aggregations
* Interview-style problem solving

---

## 🗂️ Dataset Used

```sql
customers(id, name, city)
orders(id, customer_id, order_date, amount)
products(id, name, category, price)
order_items(order_id, product_id, quantity)
employees(id, name, department, salary)
```

---

# 🟢 1. Basic SQL Questions (1–10)

### 1. Customers who placed at least one order

```sql
SELECT DISTINCT c.name
FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

---

### 2. Total number of orders

```sql
SELECT COUNT(*) FROM orders;
```

---

### 3. Total revenue

```sql
SELECT SUM(amount) FROM orders;
```

---

### 4. Customers with no orders

```sql
SELECT name
FROM customers
WHERE id NOT IN (
    SELECT customer_id FROM orders
);
```

---

### 5. Highest order amount

```sql
SELECT MAX(amount) FROM orders;
```

---

### 6. Top 5 expensive products

```sql
SELECT *
FROM products
ORDER BY price DESC
LIMIT 5;
```

---

### 7. Orders per customer

```sql
SELECT customer_id, COUNT(*) total_orders
FROM orders
GROUP BY customer_id;
```

---

### 8. Avg product price by category

```sql
SELECT category, AVG(price)
FROM products
GROUP BY category;
```

---

### 9. Employees earning above average

```sql
SELECT *
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

### 10. Second highest salary

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

# 🟡 2. Intermediate SQL Questions (11–25)

### 11. Customers with >5 orders

```sql
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

---

### 12. Most ordered product

```sql
SELECT product_id, SUM(quantity) total
FROM order_items
GROUP BY product_id
ORDER BY total DESC
LIMIT 1;
```

---

### 13. Revenue per customer

```sql
SELECT customer_id, SUM(amount)
FROM orders
GROUP BY customer_id;
```

---

### 14. Latest order per customer

```sql
SELECT customer_id, MAX(order_date)
FROM orders
GROUP BY customer_id;
```

---

### 15. Customers ordering every day (last 7 days)

👉 Use date filtering + GROUP BY + COUNT(DISTINCT date)

---

### 16. Duplicate customers

```sql
SELECT name, COUNT(*)
FROM customers
GROUP BY name
HAVING COUNT(*) > 1;
```

---

### 17. Employees with same salary

```sql
SELECT salary, COUNT(*)
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

---

### 18. Items per order

```sql
SELECT order_id, SUM(quantity)
FROM order_items
GROUP BY order_id;
```

---

### 19. Customers spending >1000

```sql
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000;
```

---

### 20. Products never ordered

```sql
SELECT name
FROM products
WHERE id NOT IN (
    SELECT product_id FROM order_items
);
```

---

### 21. First order

```sql
SELECT *
FROM orders
ORDER BY order_date
LIMIT 1;
```

---

### 22. Orders by city

```sql
SELECT c.city, COUNT(o.id)
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.city;
```

---

### 23. Top customer by revenue

```sql
SELECT customer_id, SUM(amount) revenue
FROM orders
GROUP BY customer_id
ORDER BY revenue DESC
LIMIT 1;
```

---

### 24. Products ordered >100 times

```sql
SELECT product_id
FROM order_items
GROUP BY product_id
HAVING SUM(quantity) > 100;
```

---

### 25. Dept with highest avg salary

```sql
SELECT department, AVG(salary) avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

---

# 🔴 3. Advanced SQL Questions (26–40)

### 26. 3rd highest salary

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

---

### 27. Rank employees by salary

```sql
SELECT name, salary,
RANK() OVER (ORDER BY salary DESC) rnk
FROM employees;
```

---

### 28. Top 3 salaries per department

```sql
SELECT *
FROM (
    SELECT *,
           RANK() OVER (
              PARTITION BY department
              ORDER BY salary DESC
           ) rnk
    FROM employees
) t
WHERE rnk <= 3;
```

---

### 29. Running total of orders

```sql
SELECT order_date,
SUM(amount) OVER (ORDER BY order_date) running_total
FROM orders;
```

---

### 30. Previous order amount per customer

```sql
SELECT customer_id,
amount,
LAG(amount) OVER (
PARTITION BY customer_id
ORDER BY order_date
)
FROM orders;
```

---

### 31. Daily revenue growth

👉 Use `LAG()` + date grouping

---

### 32. First order > 500

👉 Use `ROW_NUMBER()` + filter

---

### 33. Product revenue contribution %

👉 Use window SUM()

---

### 34. Top selling category

👉 GROUP BY + SUM(quantity)

---

### 35. Customer retention rate

👉 Cohort + date logic

---

### 36. Products bought together

👉 Self join on order_items

---

### 37. Longest gap between orders

👉 Use `LAG()` + date diff

---

### 38. Employees earning more than manager

👉 Self join employees table

---

### 39. Median salary

👉 Use window functions

---

### 40. Monthly cumulative revenue

👉 DATE_TRUNC + window SUM

---

# 🔥 4. Hard FAANG-Level Questions (41–50)

### 41. Customers ordering on consecutive days

👉 Window + date difference

---

### 42. Most popular product per city

👉 JOIN + RANK()

---

### 43. Fraud detection (rapid orders)

👉 Timestamp diff logic

---

### 44. Sessionized user activity

👉 Gap-based grouping

---

### 45. Churned customers

👉 Last order date logic

---

### 46. Top product per category

👉 RANK() partition

---

### 47. Revenue share per category

👉 Window percentage

---

### 48. Customers who bought all products in category

👉 Division problem (very important)

---

### 49. Second purchase date

👉 ROW_NUMBER()

---

### 50. Most profitable cohort

👉 Cohort analysis

---

# 🧠 Interview Focus (Critical)

| Area             | Importance |
| ---------------- | ---------- |
| JOINs            | ⭐⭐⭐⭐⭐      |
| Window Functions | ⭐⭐⭐⭐⭐      |
| Aggregations     | ⭐⭐⭐⭐       |
| Subqueries       | ⭐⭐⭐⭐       |
| CTEs             | ⭐⭐⭐        |
| Indexing         | ⭐⭐⭐        |

---

# 💡 Pro Tips

* Always clarify schema before solving
* Think in **steps (CTEs)** for complex queries
* Optimize using **indexes**
* Use **window functions instead of subqueries** where possible
* Explain your approach clearly (very important in interviews)

---

# 🚀 What’s Next?

👉 **Day 7 — Database Design + Case Studies**

* Real-world system design problems
* Schema design
* Trade-offs & optimization
* Interview case studies (Uber, Swiggy, Amazon)

---

# ✅ Outcome of Day 6

After completing this:

* You can solve **80% of SQL interview questions**
* You understand **real business problems**
* You are ready for **FAANG-level SQL rounds**

---
