# Day 7 — Database Design Case Study (Interview Level)

*Schema Design · Normalization · Indexing · Real-world queries*

---

# Case Study — E-Commerce System (Amazon / Flipkart)

**Requirements:**
- Users register and place orders
- Orders contain multiple products
- Products belong to categories
- Payments are recorded per order
- Order status is tracked

---

## Step 1 — Entities

```
Users · Categories · Products · Orders · Order_Items · Payments
```

---

## Step 2 — Schema

```sql
CREATE TABLE users (
    user_id    INT PRIMARY KEY AUTO_INCREMENT,
    name       VARCHAR(100),
    email      VARCHAR(100) UNIQUE,
    phone      VARCHAR(15),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE categories (
    category_id   INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100)
);

CREATE TABLE products (
    product_id  INT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(200),
    price       DECIMAL(10,2),
    category_id INT,
    stock       INT,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

CREATE TABLE orders (
    order_id     INT PRIMARY KEY AUTO_INCREMENT,
    user_id      INT,
    order_date   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2),
    status       ENUM('placed','shipped','delivered','cancelled') DEFAULT 'placed',
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Junction table: resolves Orders <-> Products many-to-many
-- price stored here = snapshot at purchase time (product price can change)
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id      INT,
    product_id    INT,
    quantity      INT,
    price         DECIMAL(10,2),
    FOREIGN KEY (order_id)    REFERENCES orders(order_id),
    FOREIGN KEY (product_id)  REFERENCES products(product_id)
);

CREATE TABLE payments (
    payment_id     INT PRIMARY KEY AUTO_INCREMENT,
    order_id       INT UNIQUE,
    payment_method VARCHAR(50),
    payment_status ENUM('pending','completed','failed') DEFAULT 'pending',
    payment_date   TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

> `order_id UNIQUE` on payments prevents duplicate payment records for the same order.

---

## Step 3 — Sample Data

```sql
INSERT INTO users (name, email, phone) VALUES
('Rahul', 'rahul@gmail.com', '9999999991'),
('Amit',  'amit@gmail.com',  '9999999992'),
('Neha',  'neha@gmail.com',  '9999999993');

INSERT INTO categories (category_name) VALUES
('Electronics'), ('Clothing');

INSERT INTO products (name, price, category_id, stock) VALUES
('Laptop',  70000.00, 1, 50),
('Phone',   30000.00, 1, 100),
('T-Shirt',   500.00, 2, 200);

INSERT INTO orders (user_id, order_date, total_amount, status) VALUES
(1, '2024-01-10 10:30:00', 100000.00, 'delivered'),
(1, '2024-02-15 14:00:00',  30000.00, 'shipped'),
(2, '2024-02-20 09:00:00',    500.00, 'placed');

INSERT INTO order_items (order_id, product_id, quantity, price) VALUES
(1, 1, 1, 70000.00),
(1, 2, 1, 30000.00),
(2, 2, 1, 30000.00),
(3, 3, 1,   500.00);

INSERT INTO payments (order_id, payment_method, payment_status, payment_date) VALUES
(1, 'Credit Card', 'completed', '2024-01-10 10:35:00'),
(2, 'UPI',         'pending',   NULL),
(3, 'COD',         'pending',   NULL);
```

---

## Step 4 — Interview Queries

### 1. Total orders per user (include users with zero orders)

```sql
SELECT u.name, COUNT(o.order_id) AS total_orders
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.name;
```

---

### 2. Top 5 most sold products

```sql
SELECT p.name, SUM(oi.quantity) AS total_sold
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.name
ORDER BY total_sold DESC
LIMIT 5;
```

---

### 3. Total revenue

```sql
SELECT SUM(total_amount) AS total_revenue FROM orders;
```

---

### 4. Users with no orders

```sql
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
```

---

### 5. Highest value order

```sql
SELECT * FROM orders ORDER BY total_amount DESC LIMIT 1;
```

---

### 6. Most popular category

```sql
SELECT c.category_name, SUM(oi.quantity) AS total_sales
FROM order_items oi
JOIN products p   ON oi.product_id   = p.product_id
JOIN categories c ON p.category_id   = c.category_id
GROUP BY c.category_name
ORDER BY total_sales DESC
LIMIT 1;
```

---

### 7. Orders by status

```sql
SELECT status, COUNT(*) AS count
FROM orders
GROUP BY status;
```

---

### 8. Revenue per user with their latest order date

```sql
SELECT u.name,
       SUM(o.total_amount)  AS total_spent,
       MAX(o.order_date)    AS last_order
FROM users u
JOIN orders o ON u.user_id = o.user_id
GROUP BY u.name;
```

---

## Step 5 — Index Strategy

```sql
CREATE INDEX idx_orders_user_id       ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product  ON order_items(product_id);
CREATE INDEX idx_products_category    ON products(category_id);
```

**When to add an index:**
- Column appears in `WHERE`, `JOIN ON`, or `ORDER BY`
- High cardinality columns (user_id, product_id) — good candidates
- Low cardinality (status with 4 values) — index rarely helps

**Check if index is used:**
```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 1;
-- type: ref = index used ✔   type: ALL = full scan ✗
```

---

## Step 6 — Normalization

### 1NF — Atomic values, no repeating groups

```
-- BAD: multiple products in one column
order_id | products
1        | 'phone,laptop'

-- GOOD: separate order_items table, one product per row
```

---

### 2NF — No partial dependency (applies to composite keys)

A non-key column must depend on the **entire** primary key, not just part of it.

```
-- BAD: order_items(order_id, product_id, product_name, quantity)
-- product_name depends only on product_id, not on (order_id + product_id)

-- GOOD: product_name lives in the products table
--       order_items only stores order_id, product_id, quantity, price
```

---

### 3NF — No transitive dependency

A non-key column must not depend on another non-key column.

```
-- BAD: orders(order_id, user_id, user_city, user_state)
-- user_city → user_state is a transitive dependency (non-key → non-key)

-- GOOD: move city/state to the users table
```

---

## Key Design Decision — Why store `price` in `order_items`?

Product prices change over time. The order must preserve the price at the time of purchase.

```
Today:    products.price = 70000  (Laptop)
Tomorrow: products.price = 80000  (price hike)

→ Old order_items.price must still show 70000
```

| Table         | Stores                        |
|---------------|-------------------------------|
| `products`    | Current master price          |
| `order_items` | Price snapshot at order time  |

---

---

# System Design Interview Q&A

### Q1. Design an Amazon-style Order System

**Core tables:**

```sql
users → orders → order_items → products → categories
                     ↓
                  payments
                  shipments
```

```sql
CREATE TABLE shipments (
    shipment_id   INT PRIMARY KEY AUTO_INCREMENT,
    order_id      INT,
    address       VARCHAR(255),
    shipped_at    TIMESTAMP,
    delivered_at  TIMESTAMP,
    tracking_code VARCHAR(50),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

---

### Q2. Design a Product Reviews System

```sql
CREATE TABLE reviews (
    review_id  INT PRIMARY KEY AUTO_INCREMENT,
    user_id    INT,
    product_id INT,
    rating     TINYINT CHECK (rating BETWEEN 1 AND 5),
    comment    TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, product_id),         -- one review per user per product
    FOREIGN KEY (user_id)    REFERENCES users(user_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

### Q3. Prevent Duplicate Payments

Three layers of protection:

```sql
-- Layer 1: DB constraint — one payment row per order
ALTER TABLE payments ADD CONSTRAINT uq_order UNIQUE (order_id);

-- Layer 2: idempotency key from client (generated before the request)
ALTER TABLE payments ADD COLUMN idempotency_key VARCHAR(100) UNIQUE;

-- Layer 3: check before inserting
INSERT INTO payments (order_id, payment_method, idempotency_key)
SELECT 1, 'UPI', 'uuid-abc-123'
WHERE NOT EXISTS (
    SELECT 1 FROM payments WHERE idempotency_key = 'uuid-abc-123'
);
```

---

### Q4. Handle High Read Traffic

| Technique | What it solves |
|-----------|---------------|
| Indexes | Slow queries on large tables |
| Read Replicas | Route SELECT queries away from the primary DB |
| Redis Cache | Cache hot data (product catalog, user sessions) |
| Table Partitioning | Split large tables by date/range — e.g., orders by year |
| Archiving | Move orders older than 2 years to an archive table |

```sql
-- Example: partition orders table by year
CREATE TABLE orders (
    order_id     INT,
    order_date   DATE,
    total_amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);
```

---

### Q5. Design a Cart System

```sql
CREATE TABLE cart (
    cart_id    INT PRIMARY KEY AUTO_INCREMENT,
    user_id    INT UNIQUE,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE cart_items (
    id         INT PRIMARY KEY AUTO_INCREMENT,
    cart_id    INT,
    product_id INT,
    quantity   INT DEFAULT 1,
    UNIQUE (cart_id, product_id),
    FOREIGN KEY (cart_id)    REFERENCES cart(cart_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

> `user_id UNIQUE` on cart — one active cart per user. On checkout: copy cart_items → order_items, then clear cart.

---

---

# Practice — Food Delivery System (Swiggy / Zomato)

## Schema

```sql
CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY AUTO_INCREMENT,
    name          VARCHAR(100),
    city          VARCHAR(50),
    rating        DECIMAL(3,1)
);

CREATE TABLE menu_items (
    item_id       INT PRIMARY KEY AUTO_INCREMENT,
    restaurant_id INT,
    name          VARCHAR(100),
    price         DECIMAL(10,2),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

CREATE TABLE delivery_agents (
    agent_id INT PRIMARY KEY AUTO_INCREMENT,
    name     VARCHAR(100),
    phone    VARCHAR(15),
    status   ENUM('available','busy') DEFAULT 'available'
);

CREATE TABLE food_orders (
    order_id      INT PRIMARY KEY AUTO_INCREMENT,
    user_id       INT,
    restaurant_id INT,
    agent_id      INT,
    order_time    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    delivery_time TIMESTAMP,
    status        ENUM('placed','preparing','out_for_delivery','delivered','cancelled'),
    total_amount  DECIMAL(10,2),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id),
    FOREIGN KEY (agent_id)      REFERENCES delivery_agents(agent_id)
);

CREATE TABLE food_order_items (
    id       INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    item_id  INT,
    quantity INT,
    price    DECIMAL(10,2),
    FOREIGN KEY (order_id) REFERENCES food_orders(order_id),
    FOREIGN KEY (item_id)  REFERENCES menu_items(item_id)
);
```

## 3 Key Queries

```sql
-- 1. Top restaurants by orders
SELECT r.name, COUNT(fo.order_id) AS total_orders
FROM restaurants r
JOIN food_orders fo ON r.restaurant_id = fo.restaurant_id
GROUP BY r.name
ORDER BY total_orders DESC
LIMIT 5;

-- 2. Average delivery time per restaurant (in minutes)
SELECT r.name,
       ROUND(AVG(TIMESTAMPDIFF(MINUTE, fo.order_time, fo.delivery_time)), 1) AS avg_delivery_min
FROM food_orders fo
JOIN restaurants r ON fo.restaurant_id = r.restaurant_id
WHERE fo.delivery_time IS NOT NULL
GROUP BY r.name;

-- 3. Most ordered item per restaurant
SELECT restaurant_id, item_id, total_qty
FROM (
    SELECT fo.restaurant_id, foi.item_id, SUM(foi.quantity) AS total_qty,
           RANK() OVER (PARTITION BY fo.restaurant_id ORDER BY SUM(foi.quantity) DESC) AS rnk
    FROM food_orders fo
    JOIN food_order_items foi ON fo.order_id = foi.order_id
    GROUP BY fo.restaurant_id, foi.item_id
) t
WHERE rnk = 1;
```

---

## What Interviewers Expect (Senior Level)

| Topic | What to cover |
|-------|--------------|
| Schema Design | Entities, relationships (1-1, 1-N, N-N), junction tables |
| Normalization | 1NF/2NF/3NF with examples, not just definitions |
| Indexing | Which columns to index and why, using EXPLAIN |
| Design decisions | Why price is in order_items, why UNIQUE on payments |
| Scaling | Read replicas, partitioning, caching — with trade-offs |

> In senior interviews, explain your **reasoning**, not just the schema. "I added price to order_items because product prices change" scores higher than just showing the column.
