# 📅 Day 8 — LeetCode SQL 50 (Patterns + Interview Preparation)

## 🎯 Objective

Master the **Top 50 SQL questions from LeetCode** and understand the **core SQL patterns** used in real-world interviews (Amazon, Google, Flipkart, etc.).

This day focuses on:

- Pattern recognition (most important for interviews)
- Writing optimized SQL queries
- Solving problems using real-world datasets

---

# 🧠 SQL Patterns Covered

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

# 📊 Difficulty Breakdown

| Level  | Count |
| ------ | ----- |
| Easy   | 20    |
| Medium | 22    |
| Hard   | 8     |

---

# 🟢 EASY (20 Questions)

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

- Rising Temperature
- Average Time of Process per Machine
- Monthly Transactions
- Immediate Food Delivery II
- Product Sales Analysis III
- Biggest Single Number
- Customers Who Bought All Products
- Employees Reporting Count
- Primary Department
- Triangle Judgement
- Consecutive Numbers
- Product Price at Given Date
- Employees Whose Manager Left
- Last Person to Fit in Bus
- Count Salary Categories
- Exchange Seats
- Products Ordered in Period
- Group Sold Products by Date
- Managers with 5 Direct Reports
- User Activity (30 Days)
- Sales Analysis I
- Sales Analysis II

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

- Department Top Three Salaries
- Second Highest Salary
- Delete Duplicate Emails
- Reformat Department Table
- Tournament Winners
- Game Play Analysis IV
- Rank Scores
- Nth Highest Salary

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

Second Highest Salary

Rising Temperature

Consecutive Numbers

Department Top 3 Salaries

Customers Who Bought All Products

Managers with 5 Direct Reports

Exchange Seats

Monthly Transactions

Product Price at Given Date

Last Person to Fit in Bus

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

Joins

Aggregations

Window functions

Learn to identify patterns quickly

🚀 What's Next?

➡️ Day 9 — Advanced SQL Patterns
➡️ Day 10 — Real-world System Design Queries

🔗 Practice Link

LeetCode Study Plan:
https://leetcode.com/studyplan/top-sql-50/

🏁 Final Advice

If you can confidently solve:

Top 10 questions ✅

All medium-level questions ✅
