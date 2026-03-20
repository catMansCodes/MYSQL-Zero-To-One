# Top 50 SQL Interview Questions (Most Asked)

This is a curated list of 50 SQL questions based on:

- Product companies (Amazon, Flipkart, Walmart, Swiggy)
- LeetCode SQL 50
- Real Java backend interviews (8–10 yrs)

Structured as:

- 🟢 Easy (1–15)
- 🟡 Medium (16–35)
- 🔴 Hard (36–50)

Each includes problem + key idea (what interviewer checks).

## 🟢 EASY (1–15)

1. **Find all employees with salary > 50k**

👉 Tests: WHERE

```sql
SELECT * FROM employees WHERE salary > 50000;
```

2. **Get top 5 highest salaries**

👉 Tests: ORDER BY + LIMIT

```sql
SELECT * FROM employees ORDER BY salary DESC LIMIT 5;
```

3. **Count total employees**

👉 Tests: COUNT

```sql
SELECT COUNT(*) FROM employees;
```

4. **Find distinct departments**

👉 Tests: DISTINCT

```sql
SELECT DISTINCT department FROM employees;
```

5. **Find employees whose name starts with 'A'**

👉 Tests: LIKE

```sql
SELECT * FROM employees WHERE name LIKE 'A%';
```

6. **Find employees in multiple departments**

👉 Tests: IN

```sql
SELECT * FROM employees WHERE department IN ('HR','IT');
```

7. **Find employees between salary range**

👉 Tests: BETWEEN

```sql
SELECT * FROM employees WHERE salary BETWEEN 30000 AND 70000;
```

8. **Find NULL values**

👉 Tests: IS NULL

```sql
SELECT * FROM employees WHERE manager_id IS NULL;
```

9. **Sort employees by salary ascending**

👉 Tests: ORDER BY

```sql
SELECT * FROM employees ORDER BY salary ASC;
```

10. **Get employee names only**

👉 Tests: Projection

```sql
SELECT name FROM employees;
```

11. **Count employees per department**

👉 Tests: GROUP BY

```sql
SELECT department, COUNT(*) FROM employees GROUP BY department;
```

12. **Find max salary**

👉 Tests: MAX()

```sql
SELECT MAX(salary) FROM employees;
```

13. **Find min salary**

```sql
SELECT MIN(salary) FROM employees;
```

14. **Find average salary**

```sql
SELECT AVG(salary) FROM employees;
```

15. **Find total salary**

```sql
SELECT SUM(salary) FROM employees;
```

## 🟡 MEDIUM (16–35)

16. **Find second highest salary**

👉 Tests: Subquery

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

17. **Find employees earning above average**

👉 Tests: Subquery + Aggregation

```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

18. **Get employee with department name**

👉 Tests: JOIN

```sql
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;
```

19. **Find employees without department**

👉 Tests: LEFT JOIN + NULL

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL;
```

20. **Count employees in each department having >5 employees**

👉 Tests: HAVING

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

21. **Find duplicate records**

👉 Tests: GROUP BY

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

22. **Delete duplicate records**

👉 Tests: Self join

```sql
DELETE u1
FROM users u1
JOIN users u2
ON u1.email = u2.email AND u1.id > u2.id;
```

23. **Find top 3 salaries per department**

👉 Tests: Window function

```sql
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) rn
    FROM employees
) t
WHERE rn <= 3;
```

24. **Rank employees**

👉 Tests: RANK()

```sql
SELECT name, salary,
RANK() OVER (ORDER BY salary DESC) rank
FROM employees;
```

25. **Difference between RANK and DENSE_RANK (conceptual)**

👉 Interview expects explanation with example.

26. **Find employees with same salary**

👉 Tests: Self join

```sql
SELECT e1.name, e2.name
FROM employees e1, employees e2
WHERE e1.salary = e2.salary AND e1.id != e2.id;
```

27. **Find nth highest salary**

👉 Tests: LIMIT + OFFSET

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

28. **Find employees hired in last 30 days**

```sql
SELECT * FROM employees
WHERE hire_date >= CURDATE() - INTERVAL 30 DAY;
```

29. **Get running total of salaries**

👉 Tests: Window function

```sql
SELECT name, salary,
SUM(salary) OVER (ORDER BY emp_id) AS running_total
FROM employees;
```

30. **Find highest salary per department**

```sql
SELECT department, MAX(salary)
FROM employees
GROUP BY department;
```

31. **Find employees who are managers**

👉 Tests: Subquery

```sql
SELECT * FROM employees
WHERE emp_id IN (SELECT manager_id FROM employees);
```

32. **Replace NULL with default value**

```sql
SELECT IFNULL(manager_id, 0) FROM employees;
```

33. **Pivot-like aggregation**

```sql
SELECT
SUM(CASE WHEN department='HR' THEN salary END) AS hr_salary
FROM employees;
```

34. **Find employees with no subordinates**

```sql
SELECT * FROM employees
WHERE emp_id NOT IN (SELECT manager_id FROM employees WHERE manager_id IS NOT NULL);
```

35. **Find gap between salaries**

```sql
SELECT salary - LAG(salary) OVER (ORDER BY salary)
FROM employees;
```

## 🔴 HARD (36–50)

36. **Find median salary**

👉 Tests: Window + ranking

37. **Find top 2 products per category**

👉 Tests: PARTITION

38. **Find consecutive login days**

👉 Tests: Window + grouping trick

39. **Find users who never ordered**

👉 Tests: LEFT JOIN

40. **Find customers with max orders**

👉 Tests: GROUP + ORDER

41. **Find duplicate rows without GROUP BY**

👉 Tests: Window function

42. **Detect gaps in sequence**

👉 Tests: LEAD/LAG

43. **Find first and last record per group**

👉 Tests: Window

44. **Recursive query (hierarchy)**

👉 Tests: CTE

45. **Find employees earning more than manager**

```sql
SELECT e.name
FROM employees e
JOIN employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

46. **Running balance (bank transactions)**
47. **Find overlapping date ranges**
48. **Top customers by revenue**
49. **Convert rows to columns (pivot)**
50. **Complex reporting query (joins + aggregation + window)**

## 🧠 Interview Strategy (Very Important)

If interviewer gives problem:

- Clarify requirement
- Write basic query
- Optimize (index, window function)
- Explain edge cases

### 🔥 Most Repeated Questions (Focus These)

If short on time, prioritize:

- Second highest salary
- Top N per group
- Duplicate records
- Join-based queries
- Window functions
- Running totals
- Self join problems
