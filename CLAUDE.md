# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A structured MySQL 8.0 learning curriculum for Java full-stack developers preparing for backend/product-company interviews. Content is organized as day-by-day concept files (`day-0X-Concept.md`) containing SQL theory, sample datasets, and practice queries — all inline in Markdown as fenced SQL code blocks. There are no runnable scripts or application code.

## Repository Structure

Each day is a folder with a single `day-0X-Concept.md`:

| Folder | Topic |
|--------|-------|
| Day 1 – SQL Fundamentals | SELECT, WHERE, ORDER BY, DISTINCT, LIMIT, NULL handling |
| Day 2 - SQL Aggregation Queries | COUNT/SUM/AVG/MIN/MAX, GROUP BY, HAVING |
| Day 3 - SQL JOINs | INNER/LEFT/RIGHT/FULL/SELF JOIN, 30 interview Qs |
| Day 4 - Advanced SQL | Subqueries (scalar, correlated), Window Functions (ROW_NUMBER, RANK, DENSE_RANK, LEAD/LAG) |
| Day 5 - Indexes, Views, Stored Procedures | Indexes, composite indexes, EXPLAIN, Views, Stored Procs |
| Day 6 - Top 50 Real SQL Interview Questions | Real interview Q&A |
| Day 7 - Database Design Case Study | Schema design, normalization (1NF–3NF), e-commerce/banking case studies |
| Day 8 - LeetCode SQL Patterns | LeetCode Top 50 SQL patterns |
| Day 9 - Top Interview Question & cheatsheet | Last-hour review, theory Q&A, practice Q&A, PDF cheatsheets |

## Content Conventions

- **Sample datasets** are defined at the top of each concept file using `CREATE TABLE` + `INSERT` blocks — always MySQL 8.0 syntax.
- **Practice questions** are asked in plain English then answered with a SQL block below.
- The `employees` table (id, name, department, salary, city) is the canonical dataset reused across Day 1–4.
- Day 7+ introduces multi-table schemas (users, orders, products, payments, categories).
- No external database connection is required to work in this repo — all SQL lives in Markdown.

## MySQL 8.0 Specifics to Keep in Mind

- Window functions require MySQL 8.0+ (`OVER`, `PARTITION BY`, `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LEAD()`, `LAG()`).
- Use `WITH` (CTE) syntax where it improves readability — supported in MySQL 8.0.
- `FULL OUTER JOIN` is not natively supported; emulate with `LEFT JOIN UNION RIGHT JOIN`.
- Prefer `EXPLAIN` / `EXPLAIN ANALYZE` for performance analysis examples.
