# 📅 Day 7 — Database Design Case Study (Interview Level)

## 🎯 Objective

Master **Database Design for Real-World Systems** — a critical skill for **senior backend interviews (8–10 years experience)**.

You will learn:

* Schema Design
* Relationships (1-1, 1-N, N-N)
* Normalization (1NF, 2NF, 3NF)
* Indexing Strategy
* Real-world query solving
* System design thinking

---

# 🧠 Case Study — E-Commerce System (Amazon / Flipkart Type)

## 📌 Problem Statement

Design a system where:

* Users can register
* Users can place orders
* Orders contain multiple products
* Products belong to categories
* Payments are recorded
* Order status is tracked

---

# 🏗️ Step 1 — Identify Entities

```
Users
Products
Categories
Orders
Order_Items
Payments
```

---

# 🗂️ Step 2 — Database Schema

## 👤 Users Table

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(15),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🏷️ Categories Table

```sql
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100)
);
```

---

## 📦 Products Table

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200),
    price DECIMAL(10,2),
    category_id INT,
    stock INT,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);
```

---

## 🧾 Orders Table

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    order_date TIMESTAMP,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

---

## 🔗 Order Items Table (Many-to-Many)

```sql
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

## 💳 Payments Table

```sql
CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    payment_method VARCHAR(50),
    payment_status VARCHAR(50),
    payment_date TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

---

# 🧪 Step 3 — Dummy Data

```sql
INSERT INTO users (name, email, phone)
VALUES
('Rahul', 'rahul@gmail.com', '9999999991'),
('Amit', 'amit@gmail.com', '9999999992');

INSERT INTO categories (category_name)
VALUES
('Electronics'),
('Clothing');

INSERT INTO products (name, price, category_id, stock)
VALUES
('Laptop', 70000, 1, 50),
('Phone', 30000, 1, 100),
('T-Shirt', 500, 2, 200);
```

---

# 💼 Step 4 — Interview Queries

## 1️⃣ Total Orders per User

```sql
SELECT 
    u.name,
    COUNT(o.order_id) AS total_orders
FROM users u
LEFT JOIN orders o 
ON u.user_id = o.user_id
GROUP BY u.name;
```

---

## 2️⃣ Top 5 Most Sold Products

```sql
SELECT 
    p.name,
    SUM(oi.quantity) AS total_sold
FROM order_items oi
JOIN products p
ON oi.product_id = p.product_id
GROUP BY p.name
ORDER BY total_sold DESC
LIMIT 5;
```

---

## 3️⃣ Total Revenue

```sql
SELECT SUM(total_amount) AS total_revenue
FROM orders;
```

---

## 4️⃣ Users with No Orders

```sql
SELECT u.name
FROM users u
LEFT JOIN orders o
ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
```

---

## 5️⃣ Highest Value Order

```sql
SELECT *
FROM orders
ORDER BY total_amount DESC
LIMIT 1;
```

---

## 6️⃣ Most Popular Category

```sql
SELECT 
    c.category_name,
    SUM(oi.quantity) AS total_sales
FROM order_items oi
JOIN products p 
ON oi.product_id = p.product_id
JOIN categories c 
ON p.category_id = c.category_id
GROUP BY c.category_name
ORDER BY total_sales DESC
LIMIT 1;
```

---

# ⚡ Step 5 — Index Optimization

```sql
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_product_id ON order_items(product_id);
CREATE INDEX idx_category ON products(category_id);
```

### 💡 Interview Tip:

Indexes reduce **full table scans** and improve query performance.

---

# 📚 Step 6 — Normalization

## ✅ 1NF (First Normal Form)

* No repeating groups

❌ Bad:

```
order_id | products
1        | phone,laptop
```

✅ Good:

```
Use order_items table
```

---

## ✅ 2NF (Second Normal Form)

* All columns depend on full primary key

---

## ✅ 3NF (Third Normal Form)

* Remove transitive dependency

Example:

```
user_id | city | state
```

👉 Move `state` to separate table

---

# 🎯 Important Interview Concepts

## 🔥 Why store price in `order_items`?

Because product price can change.

Example:

```
Today: Laptop = 70000
Tomorrow: Laptop = 80000
```

Old order must retain original price.

---

## 🔥 Difference

| Table       | Purpose                   |
| ----------- | ------------------------- |
| products    | Master data               |
| order_items | Snapshot at purchase time |

---

# 🧠 System Design Interview Questions

## Q1: Design Amazon Order System

Tables:

```
Users
Orders
Order_Items
Products
Payments
Shipment
```

---

## Q2: Design Product Reviews System

```sql
CREATE TABLE reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    product_id INT,
    rating INT,
    comment TEXT,
    created_at TIMESTAMP
);
```

---

## Q3: Prevent Duplicate Payments

Solutions:

```
UNIQUE(order_id)
transaction_id
idempotency_key
```

---

## Q4: Handle High Traffic

```
Indexes
Caching (Redis)
Read Replicas
Partitioning
```

---

## Q5: Design Cart System

```
cart
cart_items
```

---

# 🚀 Scaling Strategy (Senior-Level Answer)

```
1. Read Replicas
2. Table Partitioning
3. Caching Layer
4. Archiving Old Data
5. Microservices DB Split
```

---

# 🧪 Practice Task

## 🍔 Design Food Delivery System (Swiggy / Zomato)

### Entities:

```
users
restaurants
menu_items
orders
order_items
delivery_agents
payments
```

### Your Task:

* Design schema
* Define relationships
* Add indexes
* Write 3 queries

---

# 🧭 What Interviewers Expect (8–10 Years)

You should confidently explain:

```
✔ Schema Design
✔ Relationships
✔ Indexing
✔ Normalization
✔ Performance Optimization
✔ Real-world trade-offs
```

---

💡 **Pro Tip:**
Database design questions are **deciding rounds** in senior interviews. Practice explaining your thought process clearly.

---
