## Day 1 - SQL Fundamentals (Core Querying)

### Document purpose
This document is a beginner-friendly introduction to SQL and MySQL 8.0. You will learn the basic commands, theory (DDL/DML/DCL/TCL/DQL), common table operations, constraints (PRIMARY KEY and FOREIGN KEY), and core database concepts like normalization, indexes, views, stored procedures, and transactions.

### Prerequisites
1. Basic understanding of variables/collections (in any programming language).
2. You should be able to run a command in PowerShell (or Command Prompt).
3. Recommended tool: **MySQL Workbench** (optional but helpful for visual exploration).

---

## 1) Basic about SQL and MySQL

### What is SQL?
SQL (**Structured Query Language**) is a standard language used to interact with relational databases.
Common tasks with SQL:
- Define structures (tables, columns, constraints) -> **DDL**
- Add/change/remove data -> **DML**
- Control access/permissions -> **DCL**
- Manage transaction boundaries -> **TCL**

### What is a Database?
A database is an organized collection of data, usually stored in tables (rows and columns).

### What is MySQL?
MySQL is a widely used **relational DBMS** (Database Management System).
In MySQL, the typical hierarchy is:
- **Server** -> runs MySQL
- **Database (Schema)** -> collection of tables
- **Table** -> rows/columns
- **Row** -> a record
- **Column** -> a field with a data type

### Key terms (must know)
- **Primary Key (PK)**: uniquely identifies a row.
- **Foreign Key (FK)**: references a row in another table.
- **Row** vs **Column**
- **Null**: means "no value" (not the same as empty string or 0)

---

## 2) MySQL 8.0 Download and Installation (Windows - general steps)

### Download
1. Go to the official MySQL website: https://dev.mysql.com/downloads/mysql/
2. Download **MySQL Installer for Windows** (usually the easiest option).
3. Choose **MySQL Community Server**.

### Install (high-level)
1. Run the installer.
2. Select products (commonly **MySQL Server 8.0** + **MySQL Workbench**).
3. During setup, you will typically set:
   - Root password (save it)
   - Port (default often 3306)
4. After install:
   - Confirm MySQL Server starts successfully

### Verify installation (PowerShell)
Run:
```sql
mysql --version;
```
If `mysql` command is not found, it usually means MySQL client tools are not in your PATH.
As an alternative, you can verify using Workbench connection or the MySQL service status.

### Connect using Workbench (recommended)
1. Open MySQL Workbench
2. Create a new connection
3. Use:
   - Hostname: `localhost`
   - Port: `3306`
   - Username: `root`
4. Test connection

---

## 3) Database commands with examples

### Show databases
```sql
SHOW DATABASES;
```

### Create database
```sql
CREATE DATABASE school_db;
```
Optionally specify character set:
```sql
CREATE DATABASE school_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
```

### Use database
```sql
USE school_db;
```

### Show create database
```sql
SHOW CREATE DATABASE school_db;
```

### Alter database
In MySQL, you can alter database defaults like charset/collation.
Example (may depend on MySQL version/config):
```sql
ALTER DATABASE school_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
```

### Drop database
```sql
DROP DATABASE school_db;
```
> `DROP DATABASE` deletes the database and all tables inside it.

---

## 4) DDL, DML, DCL, TCL, DQL + MySQL data types (theory + examples)

### DDL (Data Definition Language)
Used to define or change structure: databases, tables, indexes, views.
Common DDL:
- `CREATE`, `ALTER`, `DROP`, `TRUNCATE` (TRUNCATE is sometimes treated separately)

Example:
```sql
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(100)
);
```

### DML (Data Manipulation Language)
Used to change data inside tables.
Common DML:
- `INSERT`, `UPDATE`, `DELETE`

Example:
```sql
INSERT INTO students (id, name) VALUES (1, 'Ali');
UPDATE students SET name = 'Aly' WHERE id = 1;
DELETE FROM students WHERE id = 1;
```

### DQL (Data Query Language)
Used to fetch data. In practice, this mainly means:
- `SELECT`

Example:
```sql
SELECT * FROM students;
SELECT name FROM students WHERE id = 1;
```

### DCL (Data Control Language)
Used for permissions/authorization.
Common DCL:
- `GRANT`, `REVOKE`

Example:
```sql
GRANT SELECT ON school_db.* TO 'app_user'@'localhost';
REVOKE SELECT ON school_db.* FROM 'app_user'@'localhost';
```
> DCL needs proper user privileges (and is server/admin level).

### TCL (Transaction Control Language)
Controls transactions:
- `COMMIT`, `ROLLBACK`, `START TRANSACTION` (and `SAVEPOINT` in advanced usage)

Example:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

## MySQL Data Types (common ones)

### Numeric types
- `INT` / `INTEGER`
- `BIGINT`
- `DECIMAL(M, D)` for money/precise values (recommended for currency)
- `FLOAT`, `DOUBLE` for approximate floating numbers
- `UNSIGNED` (only non-negative numbers)

Example:
```sql
price DECIMAL(10,2)
age INT UNSIGNED
```

### String types
- `CHAR(n)` fixed-length
- `VARCHAR(n)` variable-length (most common)
- `TEXT` / `LONGTEXT` for large text
- `ENUM('a','b',...)` allowed fixed values

Example:
```sql
name VARCHAR(100),
bio TEXT
```

### Date and time types
- `DATE` (YYYY-MM-DD)
- `DATETIME` (date + time)
- `TIMESTAMP` (date + time, also influenced by time zone settings)
- `TIME`

Example:
```sql
birth_date DATE,
created_at TIMESTAMP
```

### Other useful types
- `BOOLEAN` is typically a synonym for `TINYINT(1)`
- `JSON` for storing JSON documents (MySQL 8.0+ supports JSON type)

---

## 5) Table commands with examples

### Create table (basic)
```sql
CREATE TABLE students (
  id INT NOT NULL,
  name VARCHAR(100) NOT NULL,
  age INT,
  PRIMARY KEY (id)
);
```

### Show tables
```sql
SHOW TABLES;
```

### Describe / show columns
```sql
DESCRIBE students;
-- or
SHOW COLUMNS FROM students;
```

### Show create table
```sql
SHOW CREATE TABLE students;
```

### Alter table
Add a column:
```sql
ALTER TABLE students
ADD COLUMN email VARCHAR(150);
```

Modify a column type:
```sql
ALTER TABLE students
MODIFY COLUMN age BIGINT;
```

Rename a column (example):
```sql
ALTER TABLE students
CHANGE COLUMN email student_email VARCHAR(150);
```

Drop a column:
```sql
ALTER TABLE students
DROP COLUMN student_email;
```

### Delete table vs Truncate table (important difference)
- `DELETE` removes rows one-by-one and can be rolled back
- `TRUNCATE` removes all rows faster (usually resets auto_increment) and is often treated like DDL (limited rollback behavior depends on engine/settings)

`DELETE` example:
```sql
DELETE FROM students WHERE age < 18;
```

`TRUNCATE` example:
```sql
TRUNCATE TABLE students;
```

### Drop table
```sql
DROP TABLE students;
```

---

## 6) Complete CRUD operation example (dataset + table + insert/update/delete/select)

In this example, we will build a tiny dataset for a store:
- `products` table
- Perform Create, Read, Update, Delete operations

### Step 1: Create database + table
```sql
CREATE DATABASE demo_shop;
USE demo_shop;

CREATE TABLE products (
  product_id INT NOT NULL AUTO_INCREMENT,
  name VARCHAR(120) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  quantity INT NOT NULL DEFAULT 0,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (product_id)
) ENGINE=InnoDB;
```

### Step 2: CREATE (insert)
```sql
INSERT INTO products (name, price, quantity)
VALUES
('Laptop', 850.00, 10),
('Mouse', 25.50, 200);
```

### Step 3: READ (view data)
View all:
```sql
SELECT * FROM products;
```
Filter:
```sql
SELECT product_id, name, price
FROM products
WHERE price > 100
ORDER BY price DESC;
```
Limit:
```sql
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 1;
```

### Step 4: UPDATE (change data)
```sql
UPDATE products
SET price = 799.00, quantity = quantity + 5
WHERE name = 'Laptop';
```

### Step 5: DELETE (remove data)
```sql
DELETE FROM products
WHERE name = 'Mouse';
```

### Quick confirmation
```sql
SELECT * FROM products;
```

---

## 7) Foreign key (FK) concept + 2-table example

### What is a Foreign Key?
A foreign key is a column (or set of columns) in a child table that references a primary key in a parent table.
It helps enforce referential integrity:
- You cannot create an order that references a non-existing customer (depending on constraints)

### Example: Customers and Orders

Parent table: `customers`
Child table: `orders` (contains `customer_id` as FK)

### Step 1: Create tables
```sql
CREATE DATABASE fk_demo;
USE fk_demo;

CREATE TABLE customers (
  customer_id INT NOT NULL AUTO_INCREMENT,
  customer_name VARCHAR(100) NOT NULL,
  PRIMARY KEY (customer_id)
) ENGINE=InnoDB;

CREATE TABLE orders (
  order_id INT NOT NULL AUTO_INCREMENT,
  customer_id INT NOT NULL,
  order_date DATE NOT NULL,
  total_amount DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (order_id),
  CONSTRAINT fk_orders_customer
    FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
) ENGINE=InnoDB;
```

### Step 2: Insert data
```sql
INSERT INTO customers (customer_name) VALUES ('Ravi'), ('Sara');

INSERT INTO orders (customer_id, order_date, total_amount)
VALUES
(1, '2026-01-10', 120.00),
(2, '2026-01-11', 55.50);
```

### Step 3: Test cascade behavior
If you delete a customer, MySQL will automatically delete related orders (because of `ON DELETE CASCADE`):
```sql
DELETE FROM customers WHERE customer_id = 1;
```
Now `orders` rows referencing that customer should be gone.

---

## 8) ACID property with example

### Why ACID matters
ACID describes transaction reliability in a database:
- **A - Atomicity**: either all changes happen or none happen
- **C - Consistency**: database rules remain valid (constraints/invariants)
- **I - Isolation**: concurrent transactions do not interfere unexpectedly
- **D - Durability**: once committed, changes persist even after failure

### Example: Transfer money between accounts
We use a transaction to ensure both operations succeed.

```sql
USE demo_shop;

CREATE TABLE accounts (
  account_id INT NOT NULL AUTO_INCREMENT,
  owner VARCHAR(100) NOT NULL,
  balance DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (account_id)
) ENGINE=InnoDB;

INSERT INTO accounts (owner, balance) VALUES ('A', 500.00), ('B', 300.00);

START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE owner = 'A';
UPDATE accounts SET balance = balance + 100 WHERE owner = 'B';

-- If everything is correct:
COMMIT;

-- If something fails, you could do:
-- ROLLBACK;
```

What ACID guarantees here:
- Atomicity: if you roll back, no partial transfer remains
- Consistency: balances remain valid per constraints (and you can add checks)
- Isolation: other sessions won't see partial changes (depending on isolation level)
- Durability: committed changes stay after restart/crash

---

## 9) Normalization with example

### What is normalization?
Normalization organizes data to reduce redundancy and improve integrity.
It usually refers to:
- 1NF (First Normal Form)
- 2NF (Second Normal Form)
- 3NF (Third Normal Form)

### Example scenario
Imagine you store order data like this (bad design):
- `Orders` table contains both order header and repeated product info in one row.

#### Unnormalized concept (repeating groups)
Example (conceptual):
- Order 101 contains Product A and Product B

This may cause problems:
- Data duplication (same product details repeated)
- Update anomalies (change product price everywhere)
- Insert anomalies, delete anomalies

### Normalized approach (3NF idea)
Use separate tables:
- `orders` (order header)
- `order_items` (items inside order)
- `products` (product master data)

Benefits:
- Product data stored once
- Updates become simpler and consistent
- Foreign keys enforce relationships

### Tiny SQL example (before vs after)
#### Before (bad idea: product info duplicated inside order table)
```sql
-- Conceptual/un-normalized idea:
-- Each order row repeats product details.

CREATE TABLE orders_legacy (
  order_id INT NOT NULL,
  customer_name VARCHAR(100) NOT NULL,
  product_name VARCHAR(100) NOT NULL,
  product_price DECIMAL(10,2) NOT NULL,
  qty INT NOT NULL,
  PRIMARY KEY (order_id, product_name)
);
```

#### After (normalized: order + product + order_items)
```sql
-- Normalized design (typical 3NF shape)
CREATE TABLE orders (
  order_id INT NOT NULL AUTO_INCREMENT,
  customer_name VARCHAR(100) NOT NULL,
  order_date DATE NOT NULL,
  PRIMARY KEY (order_id)
) ENGINE=InnoDB;

CREATE TABLE products (
  product_id INT NOT NULL AUTO_INCREMENT,
  product_name VARCHAR(100) NOT NULL,
  product_price DECIMAL(10,2) NOT NULL,
  PRIMARY KEY (product_id),
  UNIQUE KEY uk_products_name (product_name)
) ENGINE=InnoDB;

CREATE TABLE order_items (
  order_id INT NOT NULL,
  product_id INT NOT NULL,
  qty INT NOT NULL,
  PRIMARY KEY (order_id, product_id),
  CONSTRAINT fk_order_items_order
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
  CONSTRAINT fk_order_items_product
    FOREIGN KEY (product_id) REFERENCES products(product_id)
) ENGINE=InnoDB;
```

Why this helps:
- Updating `products.product_price` updates all future queries without editing multiple duplicated rows.
- `order_items` keeps the "many products per order" relationship clean.

---

## 10) Index with example

### Why indexes exist
Indexes speed up data retrieval, especially with `WHERE`, `JOIN`, and `ORDER BY`.
Trade-off:
- Indexes take disk space
- Indexes slow down writes (`INSERT/UPDATE/DELETE`) because indexes must be updated

### Basic concept (InnoDB)
InnoDB commonly uses B-tree indexes.
The `PRIMARY KEY` is typically used as a clustered index strategy.

### Create an index
```sql
USE demo_shop;

CREATE INDEX idx_products_name
ON products(name);
```

### Composite index (multiple columns)
Example:
```sql
CREATE INDEX idx_products_price_qty
ON products(price, quantity);
```

### Using `EXPLAIN` to see query plan
```sql
EXPLAIN
SELECT * FROM products
WHERE price > 100 AND quantity > 0;
```
Review whether MySQL uses your index (look for `key` column in results).

---

## 11) View and Stored Procedure with examples

### Views
A **view** is a virtual table defined by a `SELECT` query.
It stores the query logic, not the data itself.

Example:
```sql
USE demo_shop;

CREATE VIEW expensive_products AS
SELECT product_id, name, price, quantity
FROM products
WHERE price >= 100;
```

Query the view like a table:
```sql
SELECT * FROM expensive_products;
```

Note:
- Views may have restrictions for `INSERT/UPDATE/DELETE` depending on the query definition.

### Stored Procedures
A **stored procedure** is reusable SQL code stored in the database.

Example: procedure that returns products above a given price
```sql
USE demo_shop;

DELIMITER //

CREATE PROCEDURE GetProductsAbovePrice(IN min_price DECIMAL(10,2))
BEGIN
  SELECT product_id, name, price, quantity
  FROM products
  WHERE price > min_price
  ORDER BY price DESC;
END //

DELIMITER ;
```

Call it:
```sql
CALL GetProductsAbovePrice(50.00);
```

---

## 12) Transaction with example (explicit COMMIT / ROLLBACK)

Transactions group multiple statements into a single unit of work.
If you roll back, none of the changes in the transaction should remain.

### Example: Transfer with rollback on insufficient funds
```sql
USE demo_shop;

START TRANSACTION;

-- Suppose we transfer 100 from A to B
UPDATE accounts
SET balance = balance - 100
WHERE owner = 'A' AND balance >= 100;

-- If A had not enough funds, the first UPDATE affects 0 rows.
-- We can check affected rows (simple approach shown conceptually).
-- Exact error-handling logic can be improved with stored procedures.

-- For demonstration, assume we decide:
-- COMMIT if successful
COMMIT;

-- Or if we detect a problem:
-- ROLLBACK;
```

Practical tip:
- For real production logic, use stored procedures/functions or application code to check outcomes and decide commit/rollback.

---

## Extra important topics (quick notes)

### SQL syntax basics
- SQL keywords are case-insensitive (`SELECT` == `select`)
- Statements usually end with `;`
- Comments:
  - `-- comment until end of line`
  - `# comment`
  - `/* multi-line comment */`

### NULL handling
`NULL` means unknown/no value.
Use `IS NULL` and `IS NOT NULL`:
```sql
SELECT * FROM students WHERE email IS NULL;
```
Do not use:
```sql
WHERE email = NULL;
```

### Engines and constraints
- For foreign keys and transactions, you typically should use `ENGINE=InnoDB`.

### Collation (strings sorting rules)
Collation affects comparisons and ordering.
For modern Unicode support, prefer `utf8mb4`.

---

## Practice tasks (suggested)
1. Create a database named `my_first_db` and show it with `SHOW DATABASES`.
2. Create a table `employees` with `id`, `name`, `salary`, and a primary key.
3. Insert 5 rows and run:
   - `SELECT` with `WHERE`
   - `ORDER BY`
   - `LIMIT`
4. Perform update and delete using `WHERE`.
5. Build a foreign key example with `departments` and `employees`.
6. Create a view that shows only employees with salary above a threshold.
7. Create an index and use `EXPLAIN` to observe query plan changes.
