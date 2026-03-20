# 📅 Day 8 — LeetCode SQL 50 (Patterns + Interview Preparation)

## 🎯 Objective

Master the **Top 50 SQL questions from LeetCode** and understand the **core SQL patterns** used in real-world interviews (Amazon, Google, Flipkart, etc.).

This day focuses on:

- Pattern recognition (most important for interviews)
- Writing optimized SQL queries
- Solving problems using real-world datasets

---

## 📚 Index
1. Objective
2. SQL Patterns Covered
3. Difficulty Breakdown
4. Question Lists + Examples

## 🧠 SQL Patterns Covered

| Pattern                 | Description                  |
| ----------------------- | ---------------------------- |
| Filtering               | WHERE, LIKE, conditions      |
| Joins                   | INNER, LEFT, SELF JOIN       |
| Aggregation             | GROUP BY, HAVING             |
| Conditional Aggregation | CASE WHEN                    |
| Subqueries              | IN, EXISTS                   |
| Window Functions        | RANK, DENSE_RANK, ROW_NUMBER |
| Date Functions          | DATEDIFF                     |
| String Functions        | CONCAT, LOWER                |
| Ranking                 | Top N per group              |
| Pivoting                | CASE-based transformation    |

---

## 📊 Difficulty Breakdown

| Level  | Count |
| ------ | ----- |
| Easy   | 20    |
| Medium | 22    |
| Hard   | 8     |

---

https://leetcode.com/studyplan/top-sql-50/

## 🟢 EASY (20 Questions)

Focus: **Basic SELECT, WHERE, JOIN**

1. Recyclable and Low Fat Products
2. Find Customer Referee
3. Big Countries
4. Article Views I
5. Invalid Tweets
6. Replace Employee ID With Unique Identifier
7. Product Sales Analysis I
8. Customer Who Visited but Did Not Make Transactions
9. Employee Bonus
10. Students and Examinations
11. Not Boring Movies
12. Average Selling Price
13. Project Employees I
14. Percentage of Users Attended a Contest
15. Queries Quality and Percentage
16. Number of Unique Subjects Taught
17. Find Followers Count
18. Classes More Than 5 Students
19. Fix Names in a Table
20. Patients With a Condition

---

## ✅ Example — Big Countries

```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000
   OR population >= 25000000;
```

---

💡 Concept:

Filtering using WHERE

🟡 **MEDIUM (22 Questions)**

Focus: Joins, Aggregation, Subqueries

21. Rising Temperature
22. Average Time of Process per Machine
23. Monthly Transactions
24. Immediate Food Delivery II
25. Product Sales Analysis III
26. Biggest Single Number
27. Customers Who Bought All Products
28. Employees Reporting Count
29. Primary Department
30. Triangle Judgement
31. Consecutive Numbers
32. Product Price at Given Date
33. Employees Whose Manager Left
34. Last Person to Fit in Bus
35. Count Salary Categories
36. Exchange Seats
37. Products Ordered in Period
38. Group Sold Products by Date
39. Managers with 5 Direct Reports
40. User Activity (30 Days)
41. Sales Analysis I
42. Sales Analysis II

✅ **Example — Rising Temperature**

```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
AND w1.temperature > w2.temperature;
```

💡 Concepts:

- Self Join
- Date comparison

🔴 **HARD (8 Questions)**

Focus: Window Functions + Advanced Logic

43. Department Top Three Salaries
44. Second Highest Salary
45. Delete Duplicate Emails
46. Reformat Department Table
47. Tournament Winners
48. Game Play Analysis IV
49. Rank Scores
50. Nth Highest Salary

✅ **Example — Department Top 3 Salaries**

```sql
WITH Ranked AS (
  SELECT
      e.name,
      e.salary,
      e.departmentId,
      DENSE_RANK() OVER (
          PARTITION BY departmentId
          ORDER BY salary DESC
      ) AS rnk
  FROM Employee e
)

SELECT d.name AS Department,
       r.name AS Employee,
       r.salary
FROM Ranked r
JOIN Department d
ON r.departmentId = d.id
WHERE r.rnk <= 3;
```

💡 Concepts:

- Window Functions
- Ranking per group

⭐ Top 10 Must-Do Interview Questions

These cover 80% of SQL interview patterns:

1. Second Highest Salary
2. Rising Temperature
3. Consecutive Numbers
4. Department Top 3 Salaries
5. Customers Who Bought All Products
6. Managers with 5 Direct Reports
7. Exchange Seats
8. Monthly Transactions
9. Product Price at Given Date
10. Last Person to Fit in Bus

🧩 **SQL Pattern Cheat Sheet**

| Problem Type         | Solution Pattern        |
| -------------------- | ----------------------- |
| Second highest value | ORDER BY + LIMIT OFFSET |
| Top N per group      | DENSE_RANK()            |
| Consecutive rows     | LAG / LEAD              |
| Running totals       | SUM() OVER              |
| Missing records      | LEFT JOIN + IS NULL     |
| Pivot data           | CASE WHEN               |

⏱️ **Practice Plan**

|   Step | Task        |      Time |
| -----: | ----------- | --------: |
| Step 1 | Easy (20)   |    1 hour |
| Step 2 | Medium (22) | 2–3 hours |
| Step 3 | Hard (8)    | 1–2 hours |

📌 Key Takeaways

SQL interviews are pattern-based, not syntax-based

Focus on:

- Joins
- Aggregations
- Window functions
- Learn to identify patterns quickly
