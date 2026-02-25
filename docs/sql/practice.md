# Thực hành SQL

Trang này tổng hợp các bài tập thực hành SQL từ cơ bản đến nâng cao.

## Database mẫu

### Schema

```sql
-- Tạo database
CREATE DATABASE ecommerce;
USE ecommerce;

-- Bảng customers
CREATE TABLE customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    city VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Bảng categories
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);

-- Bảng products
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(12, 2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- Bảng orders
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total DECIMAL(12, 2) DEFAULT 0,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- Bảng order_items
CREATE TABLE order_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(12, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

-- Bảng employees (cho bài tập hierarchy)
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(12, 2),
    manager_id INT,
    hire_date DATE,
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);
```

### Sample Data

```sql
-- Insert categories
INSERT INTO categories (name, parent_id) VALUES
('Electronics', NULL),
('Phones', 1),
('Laptops', 1),
('Accessories', 1),
('Fashion', NULL),
('Men', 5),
('Women', 5);

-- Insert products
INSERT INTO products (name, price, stock, category_id) VALUES
('iPhone 15 Pro', 28990000, 50, 2),
('Samsung Galaxy S24', 22990000, 30, 2),
('MacBook Pro 14', 49990000, 20, 3),
('Dell XPS 15', 35990000, 15, 3),
('AirPods Pro', 5990000, 100, 4),
('Samsung Buds', 3990000, 80, 4),
('Laptop Bag', 890000, 200, 4),
('Phone Case', 290000, 500, 4);

-- Insert customers
INSERT INTO customers (name, email, phone, city) VALUES
('Nguyen Van A', 'nguyenvana@email.com', '0901234567', 'Ha Noi'),
('Tran Thi B', 'tranthib@email.com', '0912345678', 'HCM'),
('Le Van C', 'levanc@email.com', '0923456789', 'Da Nang'),
('Pham Thi D', 'phamthid@email.com', '0934567890', 'Ha Noi'),
('Hoang Van E', 'hoangvane@email.com', '0945678901', 'HCM');

-- Insert orders
INSERT INTO orders (customer_id, order_date, status, total) VALUES
(1, '2024-01-15', 'delivered', 34980000),
(1, '2024-02-20', 'delivered', 5990000),
(2, '2024-01-18', 'delivered', 22990000),
(2, '2024-03-10', 'shipped', 890000),
(3, '2024-02-25', 'delivered', 49990000),
(4, '2024-03-05', 'processing', 28990000),
(5, '2024-03-15', 'pending', 3990000);

-- Insert order_items
INSERT INTO order_items (order_id, product_id, quantity, price) VALUES
(1, 1, 1, 28990000),
(1, 5, 1, 5990000),
(2, 5, 1, 5990000),
(3, 2, 1, 22990000),
(4, 7, 1, 890000),
(5, 3, 1, 49990000),
(6, 1, 1, 28990000),
(7, 6, 1, 3990000);
```

---

## Bài tập cơ bản

### SELECT & WHERE

!!! question "Bài 1"
    Lấy danh sách tất cả sản phẩm có giá > 10 triệu, sắp xếp theo giá giảm dần.

??? success "Đáp án"
    ```sql
    SELECT * FROM products
    WHERE price > 10000000
    ORDER BY price DESC;
    ```

!!! question "Bài 2"
    Tìm khách hàng ở Hà Nội hoặc HCM.

??? success "Đáp án"
    ```sql
    SELECT * FROM customers
    WHERE city IN ('Ha Noi', 'HCM');
    ```

!!! question "Bài 3"
    Tìm sản phẩm có tên chứa "Pro" hoặc "Galaxy".

??? success "Đáp án"
    ```sql
    SELECT * FROM products
    WHERE name LIKE '%Pro%' OR name LIKE '%Galaxy%';
    ```

!!! question "Bài 4"
    Lấy 5 sản phẩm đắt nhất.

??? success "Đáp án"
    ```sql
    SELECT * FROM products
    ORDER BY price DESC
    LIMIT 5;
    ```

---

### JOIN

!!! question "Bài 5"
    Lấy danh sách đơn hàng kèm tên khách hàng.

??? success "Đáp án"
    ```sql
    SELECT o.id, o.order_date, o.total, o.status, c.name AS customer_name
    FROM orders o
    INNER JOIN customers c ON o.customer_id = c.id
    ORDER BY o.order_date DESC;
    ```

!!! question "Bài 6"
    Lấy chi tiết đơn hàng: tên khách hàng, tên sản phẩm, số lượng, giá.

??? success "Đáp án"
    ```sql
    SELECT 
        c.name AS customer_name,
        p.name AS product_name,
        oi.quantity,
        oi.price,
        oi.quantity * oi.price AS subtotal
    FROM order_items oi
    INNER JOIN orders o ON oi.order_id = o.id
    INNER JOIN customers c ON o.customer_id = c.id
    INNER JOIN products p ON oi.product_id = p.id
    ORDER BY o.order_date DESC;
    ```

!!! question "Bài 7"
    Tìm sản phẩm chưa được mua lần nào.

??? success "Đáp án"
    ```sql
    SELECT p.*
    FROM products p
    LEFT JOIN order_items oi ON p.id = oi.product_id
    WHERE oi.id IS NULL;
    ```

!!! question "Bài 8"
    Lấy sản phẩm kèm tên category (bao gồm cả sản phẩm chưa có category).

??? success "Đáp án"
    ```sql
    SELECT p.name AS product_name, p.price, c.name AS category_name
    FROM products p
    LEFT JOIN categories c ON p.category_id = c.id;
    ```

---

### GROUP BY & HAVING

!!! question "Bài 9"
    Đếm số sản phẩm trong mỗi category.

??? success "Đáp án"
    ```sql
    SELECT c.name AS category, COUNT(p.id) AS product_count
    FROM categories c
    LEFT JOIN products p ON c.id = p.category_id
    GROUP BY c.id, c.name
    ORDER BY product_count DESC;
    ```

!!! question "Bài 10"
    Tính tổng doanh thu theo tháng trong năm 2024.

??? success "Đáp án"
    ```sql
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        COUNT(*) AS order_count,
        SUM(total) AS revenue
    FROM orders
    WHERE YEAR(order_date) = 2024 AND status != 'cancelled'
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
    ORDER BY month;
    ```

!!! question "Bài 11"
    Tìm khách hàng có tổng mua > 30 triệu.

??? success "Đáp án"
    ```sql
    SELECT 
        c.id,
        c.name,
        COUNT(o.id) AS order_count,
        SUM(o.total) AS total_spent
    FROM customers c
    INNER JOIN orders o ON c.id = o.customer_id
    WHERE o.status != 'cancelled'
    GROUP BY c.id, c.name
    HAVING SUM(o.total) > 30000000
    ORDER BY total_spent DESC;
    ```

!!! question "Bài 12"
    Tìm category có doanh thu > 20 triệu.

??? success "Đáp án"
    ```sql
    SELECT 
        c.name AS category,
        SUM(oi.quantity * oi.price) AS revenue
    FROM categories c
    INNER JOIN products p ON c.id = p.category_id
    INNER JOIN order_items oi ON p.id = oi.product_id
    INNER JOIN orders o ON oi.order_id = o.id
    WHERE o.status != 'cancelled'
    GROUP BY c.id, c.name
    HAVING SUM(oi.quantity * oi.price) > 20000000
    ORDER BY revenue DESC;
    ```

---

## Bài tập nâng cao

### Subquery

!!! question "Bài 13"
    Tìm sản phẩm có giá cao hơn giá trung bình của category đó.

??? success "Đáp án"
    ```sql
    SELECT p.*
    FROM products p
    WHERE p.price > (
        SELECT AVG(p2.price)
        FROM products p2
        WHERE p2.category_id = p.category_id
    );
    ```

!!! question "Bài 14"
    Tìm khách hàng có tổng mua cao nhất.

??? success "Đáp án"
    ```sql
    SELECT c.*, customer_totals.total_spent
    FROM customers c
    INNER JOIN (
        SELECT customer_id, SUM(total) AS total_spent
        FROM orders
        WHERE status != 'cancelled'
        GROUP BY customer_id
        ORDER BY total_spent DESC
        LIMIT 1
    ) customer_totals ON c.id = customer_totals.customer_id;
    ```

!!! question "Bài 15"
    Tìm đơn hàng có giá trị cao hơn trung bình của khách hàng đó.

??? success "Đáp án"
    ```sql
    SELECT o.*
    FROM orders o
    WHERE o.total > (
        SELECT AVG(o2.total)
        FROM orders o2
        WHERE o2.customer_id = o.customer_id
    );
    ```

---

### Window Functions

!!! question "Bài 16"
    Xếp hạng sản phẩm theo giá trong mỗi category.

??? success "Đáp án"
    ```sql
    SELECT 
        p.name,
        c.name AS category,
        p.price,
        RANK() OVER (PARTITION BY p.category_id ORDER BY p.price DESC) AS price_rank
    FROM products p
    INNER JOIN categories c ON p.category_id = c.id;
    ```

!!! question "Bài 17"
    Tính running total doanh thu theo ngày.

??? success "Đáp án"
    ```sql
    SELECT 
        order_date,
        SUM(total) AS daily_revenue,
        SUM(SUM(total)) OVER (ORDER BY order_date) AS running_total
    FROM orders
    WHERE status != 'cancelled'
    GROUP BY order_date
    ORDER BY order_date;
    ```

!!! question "Bài 18"
    So sánh doanh thu mỗi tháng với tháng trước.

??? success "Đáp án"
    ```sql
    WITH monthly_revenue AS (
        SELECT 
            DATE_FORMAT(order_date, '%Y-%m') AS month,
            SUM(total) AS revenue
        FROM orders
        WHERE status != 'cancelled'
        GROUP BY DATE_FORMAT(order_date, '%Y-%m')
    )
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
        revenue - LAG(revenue) OVER (ORDER BY month) AS diff,
        ROUND(
            (revenue - LAG(revenue) OVER (ORDER BY month)) / 
            LAG(revenue) OVER (ORDER BY month) * 100, 
            2
        ) AS growth_percent
    FROM monthly_revenue
    ORDER BY month;
    ```

!!! question "Bài 19"
    Lấy top 2 sản phẩm bán chạy nhất mỗi category.

??? success "Đáp án"
    ```sql
    WITH product_sales AS (
        SELECT 
            p.id,
            p.name,
            c.name AS category,
            SUM(oi.quantity) AS total_sold,
            ROW_NUMBER() OVER (
                PARTITION BY p.category_id 
                ORDER BY SUM(oi.quantity) DESC
            ) AS rn
        FROM products p
        INNER JOIN categories c ON p.category_id = c.id
        INNER JOIN order_items oi ON p.id = oi.product_id
        GROUP BY p.id, p.name, c.name, p.category_id
    )
    SELECT id, name, category, total_sold
    FROM product_sales
    WHERE rn <= 2;
    ```

---

### Bài tập tổng hợp

!!! question "Bài 20: Báo cáo doanh thu"
    Tạo báo cáo doanh thu theo tháng với các thông tin:
    
    - Tháng
    - Số đơn hàng
    - Doanh thu
    - Doanh thu tháng trước
    - Tăng trưởng (%)
    - Running total từ đầu năm

??? success "Đáp án"
    ```sql
    WITH monthly_stats AS (
        SELECT 
            DATE_FORMAT(order_date, '%Y-%m') AS month,
            COUNT(*) AS order_count,
            SUM(total) AS revenue
        FROM orders
        WHERE status != 'cancelled'
        GROUP BY DATE_FORMAT(order_date, '%Y-%m')
    )
    SELECT 
        month,
        order_count,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
        ROUND(
            (revenue - LAG(revenue) OVER (ORDER BY month)) / 
            NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100, 
            2
        ) AS growth_percent,
        SUM(revenue) OVER (ORDER BY month) AS ytd_revenue
    FROM monthly_stats
    ORDER BY month;
    ```

!!! question "Bài 21: Phân tích khách hàng RFM"
    Phân loại khách hàng theo RFM (Recency, Frequency, Monetary):
    
    - Recency: Số ngày từ đơn hàng gần nhất
    - Frequency: Số đơn hàng
    - Monetary: Tổng chi tiêu
    - Phân loại: VIP, Loyal, At Risk, New

??? success "Đáp án"
    ```sql
    WITH customer_rfm AS (
        SELECT 
            c.id,
            c.name,
            DATEDIFF(CURDATE(), MAX(o.order_date)) AS recency,
            COUNT(o.id) AS frequency,
            SUM(o.total) AS monetary
        FROM customers c
        LEFT JOIN orders o ON c.id = o.customer_id AND o.status != 'cancelled'
        GROUP BY c.id, c.name
    ),
    rfm_scores AS (
        SELECT 
            *,
            NTILE(5) OVER (ORDER BY recency DESC) AS r_score,
            NTILE(5) OVER (ORDER BY frequency) AS f_score,
            NTILE(5) OVER (ORDER BY monetary) AS m_score
        FROM customer_rfm
        WHERE frequency > 0
    )
    SELECT 
        id,
        name,
        recency,
        frequency,
        monetary,
        CONCAT(r_score, f_score, m_score) AS rfm_score,
        CASE 
            WHEN r_score >= 4 AND f_score >= 4 AND m_score >= 4 THEN 'VIP'
            WHEN r_score >= 3 AND f_score >= 3 THEN 'Loyal'
            WHEN r_score <= 2 AND f_score >= 3 THEN 'At Risk'
            WHEN r_score >= 4 AND f_score <= 2 THEN 'New'
            ELSE 'Regular'
        END AS segment
    FROM rfm_scores
    ORDER BY monetary DESC;
    ```

---

## Tips làm bài

1. **Đọc kỹ đề bài** - Xác định rõ input và output mong muốn
2. **Phân tích tables cần dùng** - Xác định các bảng và mối quan hệ
3. **Viết từng phần** - Bắt đầu từ SELECT đơn giản, thêm dần JOIN, WHERE, GROUP BY
4. **Test với data nhỏ** - Kiểm tra kết quả với sample data
5. **Optimize** - Sử dụng EXPLAIN để kiểm tra performance

## Tài nguyên học thêm

- [SQLZoo](https://sqlzoo.net/) - Interactive SQL tutorials
- [LeetCode SQL](https://leetcode.com/problemset/database/) - SQL problems
- [HackerRank SQL](https://www.hackerrank.com/domains/sql) - SQL challenges
- [Mode Analytics SQL Tutorial](https://mode.com/sql-tutorial/) - Comprehensive tutorial
