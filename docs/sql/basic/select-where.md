# SELECT & WHERE

SELECT là câu lệnh quan trọng nhất trong SQL, dùng để truy vấn dữ liệu từ database.

## SELECT cơ bản

### Cú pháp

```sql
SELECT column1, column2, ...
FROM table_name;
```

### Ví dụ

```sql
-- Lấy tất cả columns
SELECT * FROM products;

-- Lấy các columns cụ thể
SELECT name, price FROM products;

-- Đặt alias cho column
SELECT name AS product_name, price AS unit_price FROM products;

-- Tính toán
SELECT name, price, price * 0.9 AS discounted_price FROM products;
```

### DISTINCT - Loại bỏ trùng lặp

```sql
-- Lấy danh sách các thành phố (không trùng)
SELECT DISTINCT city FROM customers;

-- Distinct nhiều columns
SELECT DISTINCT city, country FROM customers;
```

---

## WHERE - Lọc dữ liệu

### Cú pháp

```sql
SELECT columns
FROM table_name
WHERE condition;
```

### Comparison Operators

| Operator | Mô tả | Ví dụ |
|----------|-------|-------|
| `=` | Bằng | `price = 100` |
| `<>` hoặc `!=` | Khác | `status <> 'cancelled'` |
| `>` | Lớn hơn | `price > 1000` |
| `<` | Nhỏ hơn | `stock < 10` |
| `>=` | Lớn hơn hoặc bằng | `age >= 18` |
| `<=` | Nhỏ hơn hoặc bằng | `quantity <= 100` |

```sql
-- Sản phẩm có giá > 10 triệu
SELECT * FROM products WHERE price > 10000000;

-- Đơn hàng chưa hoàn thành
SELECT * FROM orders WHERE status != 'completed';

-- Nhân viên có lương >= 15 triệu
SELECT * FROM employees WHERE salary >= 15000000;
```

---

## Logical Operators

### AND, OR, NOT

```sql
-- AND: Cả hai điều kiện đều đúng
SELECT * FROM products
WHERE price > 5000000 AND stock > 0;

-- OR: Một trong hai điều kiện đúng
SELECT * FROM products
WHERE category = 'Electronics' OR category = 'Phones';

-- NOT: Phủ định
SELECT * FROM orders
WHERE NOT status = 'cancelled';

-- Kết hợp (dùng ngoặc để rõ ràng)
SELECT * FROM products
WHERE (category = 'Electronics' OR category = 'Phones')
  AND price < 20000000
  AND stock > 0;
```

### BETWEEN

```sql
-- Giá từ 5 triệu đến 15 triệu
SELECT * FROM products
WHERE price BETWEEN 5000000 AND 15000000;

-- Tương đương với
SELECT * FROM products
WHERE price >= 5000000 AND price <= 15000000;

-- Ngày trong khoảng
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

### IN

```sql
-- Sản phẩm thuộc các category
SELECT * FROM products
WHERE category IN ('Electronics', 'Phones', 'Tablets');

-- Tương đương với
SELECT * FROM products
WHERE category = 'Electronics'
   OR category = 'Phones'
   OR category = 'Tablets';

-- NOT IN
SELECT * FROM products
WHERE category NOT IN ('Accessories', 'Cables');
```

### LIKE - Pattern matching

| Pattern | Mô tả |
|---------|-------|
| `%` | Bất kỳ chuỗi ký tự nào |
| `_` | Một ký tự bất kỳ |

```sql
-- Tên bắt đầu bằng 'iPhone'
SELECT * FROM products WHERE name LIKE 'iPhone%';

-- Tên chứa 'Pro'
SELECT * FROM products WHERE name LIKE '%Pro%';

-- Tên kết thúc bằng 'Max'
SELECT * FROM products WHERE name LIKE '%Max';

-- Tên có đúng 10 ký tự
SELECT * FROM products WHERE name LIKE '__________';

-- Email có domain gmail.com
SELECT * FROM customers WHERE email LIKE '%@gmail.com';
```

### IS NULL / IS NOT NULL

```sql
-- Sản phẩm chưa có mô tả
SELECT * FROM products WHERE description IS NULL;

-- Khách hàng có số điện thoại
SELECT * FROM customers WHERE phone IS NOT NULL;
```

---

## ORDER BY - Sắp xếp

### Cú pháp

```sql
SELECT columns
FROM table_name
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...;
```

### Ví dụ

```sql
-- Sắp xếp theo giá tăng dần (mặc định)
SELECT * FROM products ORDER BY price;
SELECT * FROM products ORDER BY price ASC;

-- Sắp xếp theo giá giảm dần
SELECT * FROM products ORDER BY price DESC;

-- Sắp xếp theo nhiều columns
SELECT * FROM products
ORDER BY category ASC, price DESC;

-- Sắp xếp theo vị trí column
SELECT name, price, stock FROM products
ORDER BY 2 DESC;  -- Sắp xếp theo column thứ 2 (price)

-- Sắp xếp theo expression
SELECT name, price, stock, price * stock AS total_value
FROM products
ORDER BY total_value DESC;
```

---

## LIMIT / OFFSET - Phân trang

### Cú pháp

```sql
-- MySQL, PostgreSQL
SELECT columns FROM table_name LIMIT count;
SELECT columns FROM table_name LIMIT count OFFSET skip;

-- SQL Server
SELECT TOP count columns FROM table_name;
```

### Ví dụ

```sql
-- Lấy 10 sản phẩm đầu tiên
SELECT * FROM products LIMIT 10;

-- Lấy 10 sản phẩm, bỏ qua 20 sản phẩm đầu (trang 3)
SELECT * FROM products LIMIT 10 OFFSET 20;

-- Top 5 sản phẩm đắt nhất
SELECT * FROM products
ORDER BY price DESC
LIMIT 5;

-- Phân trang: trang 2, mỗi trang 10 items
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 10;
```

---

## Aggregate Functions

### Các hàm tổng hợp

| Function | Mô tả |
|----------|-------|
| `COUNT()` | Đếm số rows |
| `SUM()` | Tổng |
| `AVG()` | Trung bình |
| `MIN()` | Giá trị nhỏ nhất |
| `MAX()` | Giá trị lớn nhất |

```sql
-- Đếm số sản phẩm
SELECT COUNT(*) FROM products;
SELECT COUNT(id) FROM products;

-- Đếm số sản phẩm có mô tả
SELECT COUNT(description) FROM products;

-- Đếm số category (không trùng)
SELECT COUNT(DISTINCT category) FROM products;

-- Tổng giá trị tồn kho
SELECT SUM(price * stock) AS total_inventory FROM products;

-- Giá trung bình
SELECT AVG(price) AS avg_price FROM products;

-- Giá cao nhất và thấp nhất
SELECT MIN(price) AS min_price, MAX(price) AS max_price FROM products;

-- Kết hợp nhiều hàm
SELECT 
    COUNT(*) AS total_products,
    SUM(stock) AS total_stock,
    AVG(price) AS avg_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products
WHERE category = 'Electronics';
```

---

## String Functions

```sql
-- Nối chuỗi
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- Độ dài chuỗi
SELECT name, LENGTH(name) AS name_length FROM products;

-- Chữ hoa/thường
SELECT UPPER(name), LOWER(email) FROM customers;

-- Cắt chuỗi
SELECT SUBSTRING(name, 1, 10) FROM products;

-- Thay thế
SELECT REPLACE(phone, '-', '') FROM customers;

-- Trim khoảng trắng
SELECT TRIM(name) FROM products;

-- Tìm vị trí
SELECT LOCATE('@', email) FROM customers;
```

---

## Date Functions

```sql
-- Ngày hiện tại
SELECT CURDATE(), CURRENT_DATE();

-- Thời gian hiện tại
SELECT NOW(), CURRENT_TIMESTAMP();

-- Trích xuất phần của ngày
SELECT 
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    DAY(order_date) AS day,
    DAYNAME(order_date) AS day_name
FROM orders;

-- Tính khoảng cách ngày
SELECT DATEDIFF(NOW(), order_date) AS days_ago FROM orders;

-- Cộng/trừ ngày
SELECT DATE_ADD(order_date, INTERVAL 7 DAY) AS delivery_date FROM orders;
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH) AS last_month;

-- Format ngày
SELECT DATE_FORMAT(order_date, '%d/%m/%Y') FROM orders;
```

---

## CASE Expression

```sql
-- CASE đơn giản
SELECT name, price,
    CASE
        WHEN price < 5000000 THEN 'Cheap'
        WHEN price < 15000000 THEN 'Medium'
        ELSE 'Expensive'
    END AS price_category
FROM products;

-- CASE với giá trị cụ thể
SELECT name, status,
    CASE status
        WHEN 'pending' THEN 'Đang chờ'
        WHEN 'processing' THEN 'Đang xử lý'
        WHEN 'shipped' THEN 'Đã gửi'
        WHEN 'delivered' THEN 'Đã giao'
        ELSE 'Không xác định'
    END AS status_vn
FROM orders;

-- CASE trong ORDER BY
SELECT * FROM products
ORDER BY 
    CASE category
        WHEN 'Electronics' THEN 1
        WHEN 'Phones' THEN 2
        ELSE 3
    END;
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Viết query lấy:
    
    1. Tất cả sản phẩm có giá > 10 triệu
    2. Sản phẩm có tên chứa "iPhone"
    3. Top 10 sản phẩm đắt nhất

!!! example "Bài tập 2"
    Viết query thống kê:
    
    1. Số lượng sản phẩm theo category
    2. Tổng giá trị đơn hàng trong tháng 1/2024
    3. Khách hàng có nhiều đơn hàng nhất

## Tiếp theo

- [JOIN](join.md)
