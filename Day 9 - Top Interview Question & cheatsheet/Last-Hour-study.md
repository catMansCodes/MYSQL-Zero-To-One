# 2-Hour MySQL Interview Revision Plan

**Goal:** Cover every high-frequency topic before a MySQL/SQL interview.
**Rule:** Don't skip blocks. Each block is self-contained.

---

## Datasets (Run these first — takes 2 min)

```sql
-- Dataset 1: employees (used in most questions)
CREATE TABLE employees (
    id         INT,
    name       VARCHAR(50),
    department VARCHAR(50),
    salary     INT,
    manager_id INT
);

INSERT INTO employees VALUES
(1, 'Rahul',  'Engineering', 90000,  5),
(2, 'Amit',   'HR',          60000,  6),
(3, 'Neha',   'Engineering', 95000,  5),
(4, 'Priya',  'Finance',     70000,  6),
(5, 'Karan',  'Engineering', 110000, NULL),
(6, 'Anita',  'HR',          80000,  NULL),
(7, 'Ravi',   'Engineering', 85000,  5),
(8, 'Sonia',  'Finance',     65000,  6);

-- Dataset 2: customers + orders (used in JOIN/aggregation questions)
CREATE TABLE customers (
    id   INT,
    name VARCHAR(50),
    city VARCHAR(50)
);

CREATE TABLE orders (
    id          INT,
    customer_id INT,
    order_date  DATE,
    amount      DECIMAL(10,2)
);

INSERT INTO customers VALUES
(1, 'Alice',   'Delhi'),
(2, 'Bob',     'Mumbai'),
(3, 'Charlie', 'Bangalore'),
(4, 'Diana',   'Delhi');

INSERT INTO orders VALUES
(1, 1, '2024-01-05', 1500.00),
(2, 1, '2024-02-10', 2300.00),
(3, 2, '2024-01-15',  800.00),
(4, 3, '2024-03-01', 4200.00),
(5, 1, '2024-03-20',  900.00);
```

---

---

# HOUR 1 — Core SQL + Most Asked Patterns (0:00–1:00)

---

## Block 1 — SQL Execution Order + Gotchas (0:00–0:10)

**Execution order (memorize this):**
```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

**Gotchas interviewers love:**
- `WHERE` filters rows BEFORE grouping. `HAVING` filters AFTER grouping.
- You CANNOT use a `SELECT` alias in `WHERE` (alias doesn't exist yet).
- You CAN use a `SELECT` alias in `ORDER BY`.
- `COUNT(*)` counts all rows. `COUNT(column)` skips NULLs.
- `NULL = NULL` is FALSE. Use `IS NULL` / `IS NOT NULL`.

---

## Block 2 — GROUP BY + HAVING (0:10–0:20)

**Q1: How many employees are in each department?**
```sql
SELECT department, COUNT(*) AS headcount
FROM employees
GROUP BY department;
```

**Q2: Which departments have more than 2 employees?**
```sql
SELECT department, COUNT(*) AS headcount
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

**Q3: Average salary per department, only where avg > 70000?**
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

**Q4: Highest salary per department?**
```sql
SELECT department, MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

---

## Block 3 — JOINs (0:20–0:35)

**JOIN cheatsheet:**
| JOIN | Returns |
|------|---------|
| `INNER JOIN` | Only matching rows in both tables |
| `LEFT JOIN`  | All left rows + matching right (NULL if no match) |
| `RIGHT JOIN` | All right rows + matching left |
| Self JOIN    | Same table joined to itself |

**Q5: Customers who placed at least one order?**
```sql
SELECT DISTINCT c.name
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;
```

**Q6: Customers with NO orders? (classic LEFT JOIN + NULL trick)**
```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

**Q7: Total amount spent by each customer?**
```sql
SELECT c.name, SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;
```

**Q8: Employee earning more than their manager? (Self JOIN)**
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

## Block 4 — Subqueries (0:35–0:50)

**Q9: Second highest salary?**
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Q10: Employees earning above company average salary?**
```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Q11: Customers who have placed more than 1 order?**
```sql
SELECT name
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) > 1
);
```

**Q12: Employees in the highest-paid department?**
```sql
SELECT name, department, salary
FROM employees
WHERE department = (
    SELECT department
    FROM employees
    GROUP BY department
    ORDER BY AVG(salary) DESC
    LIMIT 1
);
```

---

## Block 5 — Window Functions (0:50–1:00)

**Window function cheatsheet:**
| Function | Behavior |
|----------|----------|
| `ROW_NUMBER()` | Unique row number (no ties) |
| `RANK()` | Ties get same rank, next rank skips |
| `DENSE_RANK()` | Ties get same rank, next rank does NOT skip |
| `LAG(col, n)` | Value from n rows before |
| `LEAD(col, n)` | Value from n rows ahead |
| `SUM() OVER()` | Running total |

**Q13: Rank employees by salary within each department?**
```sql
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

**Q14: Top 3 earners per department? (use DENSE_RANK — ties included)**
```sql
SELECT name, department, salary
FROM (
    SELECT name, department, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk <= 3;
```

**Q15: Running total of order amounts?**
```sql
SELECT id, customer_id, amount,
       SUM(amount) OVER (ORDER BY id) AS running_total
FROM orders;
```

**Q16: Compare each employee's salary to the previous employee (same dept)?**
```sql
SELECT name, department, salary,
       LAG(salary) OVER (PARTITION BY department ORDER BY salary) AS prev_salary,
       salary - LAG(salary) OVER (PARTITION BY department ORDER BY salary) AS diff
FROM employees;
```

---

---

# HOUR 2 — Advanced Patterns + Theory Q&A (1:00–2:00)

---

## Block 6 — Duplicates + CTEs (1:00–1:15)

**Q17: Find duplicate names in a table?**
```sql
SELECT name, COUNT(*) AS occurrences
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```

**Q18: Delete duplicate rows, keep the one with the lowest id?**
```sql
DELETE e1
FROM employees e1
JOIN employees e2 ON e1.name = e2.name AND e1.id > e2.id;
```

**Q19: CTE — Employees earning above department average?**
```sql
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.name, e.department, e.salary, d.avg_salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

**Q20: Month-wise total revenue?**
```sql
SELECT DATE_FORMAT(order_date, '%Y-%m') AS month,
       SUM(amount) AS monthly_revenue
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;
```

---

## Block 7 — Indexes + Performance (1:15–1:25)

**When to add an index:**
- Column used in `WHERE`
- Column used in `JOIN ON`
- Column used in `ORDER BY` / `GROUP BY`

**When NOT to add an index:**
- Small tables (overhead > benefit)
- Columns with very few distinct values (e.g., gender, boolean)
- Columns that are frequently updated

**How to check if a query uses an index:**
```sql
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
```
Look for `type: ref` or `type: range` — good. `type: ALL` = full table scan = bad.

**Theory Q: Difference between clustered and non-clustered index?**
- **Clustered:** Data rows are physically stored in index order. One per table (Primary Key is clustered in InnoDB).
- **Non-clustered:** Separate structure pointing to data. Multiple allowed per table.

---

## Block 8 — Database Design Theory (1:25–1:40)

**Q: What is normalization?**
- Process of organizing a database to reduce redundancy.
- **1NF:** Each column has atomic values, no repeating groups.
- **2NF:** 1NF + no partial dependency (non-key columns depend on the entire PK).
- **3NF:** 2NF + no transitive dependency (non-key columns don't depend on other non-key columns).

**Q: Primary Key vs Foreign Key vs Unique Key?**
| Key | Rule |
|-----|------|
| Primary Key | Unique + NOT NULL. One per table. |
| Foreign Key | References a PK in another table. Enforces referential integrity. |
| Unique Key | Unique values allowed, but can be NULL. Multiple per table. |

**Q: What is a composite key?**
A key made of two or more columns together (e.g., `order_id + product_id` in an order_items table).

**Q: ACID properties?**
| Property | Meaning |
|----------|---------|
| Atomicity | All or nothing |
| Consistency | DB stays valid before and after |
| Isolation | Transactions don't interfere |
| Durability | Committed data persists |

---

## Block 9 — DELETE vs TRUNCATE vs DROP (1:40–1:45)

| Command | Removes | Rollback | WHERE clause | Resets AUTO_INCREMENT |
|---------|---------|----------|--------------|-----------------------|
| `DELETE` | Rows | Yes | Yes | No |
| `TRUNCATE` | All rows | No | No | Yes |
| `DROP` | Entire table | No | No | N/A |

```sql
DELETE FROM employees WHERE id = 1;   -- removes 1 row, can rollback
TRUNCATE TABLE employees;             -- wipes all rows, fast, no rollback
DROP TABLE employees;                 -- destroys the table completely
```

---

## Block 10 — Rapid-Fire Theory Q&A (1:45–2:00)

**Q: WHERE vs HAVING?**
- `WHERE` filters rows before grouping (cannot use aggregate functions).
- `HAVING` filters groups after `GROUP BY` (can use aggregate functions).

**Q: RANK() vs DENSE_RANK()?**
- Salaries: 100, 90, 90, 80
- `RANK()`: 1, 2, 2, 4 (skips 3)
- `DENSE_RANK()`: 1, 2, 2, 3 (no gaps)

**Q: What is a correlated subquery?**
A subquery that references a column from the outer query. Runs once per row of the outer query (slower).
```sql
SELECT name, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) FROM employees e2 WHERE e2.department = e1.department
);
```

**Q: What is a view?**
A saved SELECT query treated as a virtual table. Data is not stored — it's computed on each access.
```sql
CREATE VIEW high_earners AS
SELECT name, salary FROM employees WHERE salary > 80000;

SELECT * FROM high_earners;
```

**Q: UNION vs UNION ALL?**
- `UNION` removes duplicates (slower).
- `UNION ALL` keeps duplicates (faster).

**Q: What causes a Cartesian product?**
A `JOIN` without an `ON` condition. Every row in table A matches every row in table B.
```sql
-- BAD: missing ON condition
SELECT * FROM employees, departments;
```

**Q: What is a stored procedure?**
A named, reusable SQL block stored in the database.
```sql
DELIMITER //
CREATE PROCEDURE GetHighEarners()
BEGIN
    SELECT name, salary FROM employees WHERE salary > 80000;
END //
DELIMITER ;

CALL GetHighEarners();
```

**Q: How do you handle NULL in comparisons?**
```sql
-- Wrong
WHERE column = NULL     -- always false

-- Correct
WHERE column IS NULL
WHERE column IS NOT NULL
WHERE COALESCE(column, 0) > 5
```

---

---

---

# TOP 10 MUST-DO INTERVIEW QUESTIONS

These 10 questions cover 80% of SQL interview patterns. Covered/mapped below.

| # | Question | Status |
|---|----------|--------|
| 1 | Second Highest Salary | Q9 above |
| 2 | Rising Temperature | Below |
| 3 | Consecutive Numbers | Below |
| 4 | Department Top 3 Salaries | Q14 above |
| 5 | Customers Who Bought All Products | Below |
| 6 | Managers with 5 Direct Reports | Below |
| 7 | Exchange Seats | Below |
| 8 | Monthly Transactions | Q20 above |
| 9 | Product Price at Given Date | Below |
| 10 | Last Person to Fit in Bus | Below |

---

## Additional Datasets (for questions below)

```sql
-- Weather table (Rising Temperature)
CREATE TABLE Weather (id INT, recordDate DATE, temperature INT);
INSERT INTO Weather VALUES
(1, '2015-01-01', 10),
(2, '2015-01-02', 25),
(3, '2015-01-03', 20),
(4, '2015-01-04', 30);

-- Logs table (Consecutive Numbers)
CREATE TABLE Logs (id INT, num INT);
INSERT INTO Logs VALUES (1,1),(2,1),(3,1),(4,2),(5,1),(6,2),(7,2);

-- Product + Customer tables (Customers Who Bought All Products)
CREATE TABLE Product (product_key INT);
INSERT INTO Product VALUES (5),(6),(7);

CREATE TABLE Customer (customer_id INT, product_key INT);
INSERT INTO Customer VALUES (1,5),(2,6),(3,5),(3,6),(3,7),(1,6),(1,7);

-- Manager table (Managers with 5 Direct Reports)
CREATE TABLE Manager (id INT, name VARCHAR(50), department VARCHAR(50), managerId INT);
INSERT INTO Manager VALUES
(101,'John','A',NULL),
(102,'Dan','A',101),
(103,'James','A',101),
(104,'Amy','A',101),
(105,'Anne','A',101),
(106,'Ron','B',101);

-- Seat table (Exchange Seats)
CREATE TABLE Seat (id INT, student VARCHAR(50));
INSERT INTO Seat VALUES (1,'Abbot'),(2,'Doris'),(3,'Emerson'),(4,'Green'),(5,'Jeames');

-- Products price history (Product Price at Given Date)
CREATE TABLE ProductPrices (product_id INT, new_price INT, change_date DATE);
INSERT INTO ProductPrices VALUES
(1, 20, '2019-08-14'),
(2, 50, '2019-08-14'),
(1, 30, '2019-08-15'),
(1, 35, '2019-08-16'),
(2, 65, '2019-08-17'),
(3, 20, '2019-08-18');

-- Queue table (Last Person to Fit in Bus) — bus weight limit: 1000
CREATE TABLE Queue (person_name VARCHAR(50), weight INT, turn INT);
INSERT INTO Queue VALUES
('Alice',250,1),
('Alex',350,2),
('John',400,3),
('Marie',200,4),
('Bob',175,5);
```

---

## Q21: Rising Temperature

Find all dates where temperature was higher than the previous day.

**Pattern:** Self JOIN on `DATEDIFF` = 1, or LAG window function.

```sql
-- Approach 1: Self JOIN
SELECT w1.id
FROM Weather w1
JOIN Weather w2 ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;

-- Approach 2: Window function (cleaner)
SELECT id
FROM (
    SELECT id,
           temperature,
           LAG(temperature) OVER (ORDER BY recordDate) AS prev_temp,
           LAG(recordDate)  OVER (ORDER BY recordDate) AS prev_date,
           recordDate
    FROM Weather
) t
WHERE temperature > prev_temp
  AND DATEDIFF(recordDate, prev_date) = 1;
```

---

## Q22: Consecutive Numbers

Find numbers that appear at least 3 times consecutively.

**Pattern:** Self JOIN on consecutive IDs.

```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l2.id = l1.id + 1
JOIN Logs l3 ON l3.id = l1.id + 2
WHERE l1.num = l2.num AND l2.num = l3.num;
```

> Note: This works only when IDs are consecutive with no gaps. For gap-safe version use LAG/LEAD.

```sql
-- Gap-safe version
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT num,
           LAG(num, 1) OVER (ORDER BY id)  AS prev1,
           LAG(num, 2) OVER (ORDER BY id)  AS prev2
    FROM Logs
) t
WHERE num = prev1 AND num = prev2;
```

---

## Q23: Customers Who Bought All Products

Find customers who have purchased every product in the Product table.

**Pattern:** COUNT(DISTINCT) = total products.

```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

---

## Q24: Managers with 5 Direct Reports

Find managers who have at least 5 employees reporting directly to them.

**Pattern:** Subquery with GROUP BY + HAVING on managerId.

```sql
SELECT name
FROM Manager
WHERE id IN (
    SELECT managerId
    FROM Manager
    GROUP BY managerId
    HAVING COUNT(*) >= 5
);
```

---

## Q25: Exchange Seats

Swap seats of adjacent students. If total students is odd, the last student keeps their seat.

**Pattern:** CASE on `id % 2` (odd/even check).

```sql
SELECT
    CASE
        WHEN id % 2 = 1 AND id = (SELECT MAX(id) FROM Seat) THEN id  -- last odd row, no swap
        WHEN id % 2 = 1 THEN id + 1                                   -- odd → swap with next
        ELSE id - 1                                                    -- even → swap with prev
    END AS id,
    student
FROM Seat
ORDER BY id;
```

---

## Q26: Product Price at Given Date

Find the price of each product on `2019-08-16`. Products with no price change before that date default to `10`.

**Pattern:** Correlated subquery with `COALESCE` for default value.

```sql
SELECT
    product_id,
    COALESCE(
        (SELECT new_price
         FROM ProductPrices p2
         WHERE p2.product_id = p1.product_id
           AND p2.change_date <= '2019-08-16'
         ORDER BY change_date DESC
         LIMIT 1),
        10
    ) AS price
FROM (SELECT DISTINCT product_id FROM ProductPrices) p1;
```

---

## Q27: Last Person to Fit in Bus

The bus has a weight limit of 1000 kg. Passengers board in `turn` order. Find the last person who can board without exceeding the limit.

**Pattern:** Running SUM with window function, filter where running total <= 1000, take the last one.

```sql
SELECT person_name
FROM (
    SELECT person_name,
           SUM(weight) OVER (ORDER BY turn) AS running_weight
    FROM Queue
) t
WHERE running_weight <= 1000
ORDER BY running_weight DESC
LIMIT 1;
```

---

## Final Tip

If you go blank on a question, say:

> "Let me start with a straightforward approach using a subquery, then I can optimize it with a window function if needed."

This signals structured thinking — exactly what interviewers want.
