# Giới thiệu SQL

SQL (Structured Query Language) là ngôn ngữ tiêu chuẩn để tương tác với cơ sở dữ liệu quan hệ.

## Cơ sở dữ liệu quan hệ

### Khái niệm

**Cơ sở dữ liệu quan hệ (RDBMS)** lưu trữ dữ liệu trong các **bảng (tables)** có cấu trúc, với các **hàng (rows)** và **cột (columns)**.

```
┌─────────────────────────────────────────────┐
│                 customers                    │
├────┬──────────┬─────────────────┬───────────┤
│ id │   name   │      email      │   city    │
├────┼──────────┼─────────────────┼───────────┤
│  1 │ Nguyen A │ a@email.com     │ Ha Noi    │
│  2 │ Tran B   │ b@email.com     │ HCM       │
│  3 │ Le C     │ c@email.com     │ Da Nang   │
└────┴──────────┴─────────────────┴───────────┘
```

### Thuật ngữ

| Thuật ngữ | Mô tả |
|-----------|-------|
| **Database** | Tập hợp các bảng có liên quan |
| **Table** | Cấu trúc lưu trữ dữ liệu theo hàng và cột |
| **Row/Record** | Một bản ghi dữ liệu |
| **Column/Field** | Một thuộc tính của dữ liệu |
| **Primary Key** | Cột định danh duy nhất cho mỗi row |
| **Foreign Key** | Cột tham chiếu đến primary key của bảng khác |

---

## Các loại câu lệnh SQL

### DDL (Data Definition Language)

Định nghĩa cấu trúc database.

```sql
-- Tạo bảng
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2)
);

-- Sửa bảng
ALTER TABLE products ADD COLUMN stock INT DEFAULT 0;

-- Xóa bảng
DROP TABLE products;
```

### DML (Data Manipulation Language)

Thao tác với dữ liệu.

```sql
-- Thêm dữ liệu
INSERT INTO products (name, price) VALUES ('Laptop', 15000000);

-- Cập nhật dữ liệu
UPDATE products SET price = 14000000 WHERE id = 1;

-- Xóa dữ liệu
DELETE FROM products WHERE id = 1;

-- Truy vấn dữ liệu
SELECT * FROM products;
```

### DCL (Data Control Language)

Quản lý quyền truy cập.

```sql
-- Cấp quyền
GRANT SELECT, INSERT ON database.* TO 'user'@'localhost';

-- Thu hồi quyền
REVOKE INSERT ON database.* FROM 'user'@'localhost';
```

---

## Kiểu dữ liệu

### Số (Numeric)

| Kiểu | Mô tả | Phạm vi |
|------|-------|---------|
| `INT` | Số nguyên | -2.1 tỷ đến 2.1 tỷ |
| `BIGINT` | Số nguyên lớn | Rất lớn |
| `DECIMAL(p,s)` | Số thập phân chính xác | p chữ số, s sau dấu phẩy |
| `FLOAT` | Số thực | Xấp xỉ |

### Chuỗi (String)

| Kiểu | Mô tả |
|------|-------|
| `CHAR(n)` | Chuỗi cố định n ký tự |
| `VARCHAR(n)` | Chuỗi tối đa n ký tự |
| `TEXT` | Chuỗi dài |

### Ngày giờ (Date/Time)

| Kiểu | Mô tả | Ví dụ |
|------|-------|-------|
| `DATE` | Ngày | 2024-01-15 |
| `TIME` | Giờ | 14:30:00 |
| `DATETIME` | Ngày và giờ | 2024-01-15 14:30:00 |
| `TIMESTAMP` | Timestamp | Auto update |

---

## Tạo bảng

### Cú pháp

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);
```

### Ví dụ

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE,
    salary DECIMAL(10,2),
    department_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Constraints

| Constraint | Mô tả |
|------------|-------|
| `PRIMARY KEY` | Định danh duy nhất |
| `NOT NULL` | Không được null |
| `UNIQUE` | Giá trị duy nhất |
| `DEFAULT` | Giá trị mặc định |
| `CHECK` | Điều kiện kiểm tra |
| `FOREIGN KEY` | Tham chiếu bảng khác |

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE DEFAULT (CURRENT_DATE),
    total DECIMAL(10,2) CHECK (total >= 0),
    status VARCHAR(20) DEFAULT 'pending',
    
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

---

## INSERT - Thêm dữ liệu

### Cú pháp

```sql
-- Thêm một row
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- Thêm nhiều rows
INSERT INTO table_name (column1, column2, ...)
VALUES 
    (value1, value2, ...),
    (value1, value2, ...),
    (value1, value2, ...);
```

### Ví dụ

```sql
-- Thêm một khách hàng
INSERT INTO customers (name, email, city)
VALUES ('Nguyen Van A', 'a@email.com', 'Ha Noi');

-- Thêm nhiều sản phẩm
INSERT INTO products (name, price, stock) VALUES
    ('iPhone 15', 25000000, 50),
    ('Samsung S24', 22000000, 30),
    ('Xiaomi 14', 15000000, 100);

-- Thêm từ SELECT
INSERT INTO archived_orders (id, customer_id, total)
SELECT id, customer_id, total
FROM orders
WHERE order_date < '2023-01-01';
```

---

## UPDATE - Cập nhật dữ liệu

### Cú pháp

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

!!! warning "Cảnh báo"
    Luôn sử dụng WHERE để tránh cập nhật toàn bộ bảng!

### Ví dụ

```sql
-- Cập nhật một row
UPDATE products
SET price = 23000000
WHERE id = 1;

-- Cập nhật nhiều columns
UPDATE employees
SET salary = salary * 1.1, updated_at = NOW()
WHERE department_id = 5;

-- Cập nhật với điều kiện phức tạp
UPDATE orders
SET status = 'shipped'
WHERE status = 'processing' AND order_date < CURDATE() - INTERVAL 2 DAY;
```

---

## DELETE - Xóa dữ liệu

### Cú pháp

```sql
DELETE FROM table_name
WHERE condition;
```

!!! warning "Cảnh báo"
    Luôn sử dụng WHERE để tránh xóa toàn bộ bảng!

### Ví dụ

```sql
-- Xóa một row
DELETE FROM products WHERE id = 1;

-- Xóa với điều kiện
DELETE FROM orders WHERE status = 'cancelled';

-- Xóa toàn bộ (cẩn thận!)
DELETE FROM temp_data;

-- TRUNCATE - xóa nhanh hơn, reset auto_increment
TRUNCATE TABLE temp_data;
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Tạo database `school` với các bảng:
    
    1. `students` (id, name, email, birth_date, class_id)
    2. `classes` (id, name, teacher_id)
    3. `teachers` (id, name, subject)

!!! example "Bài tập 2"
    Thêm dữ liệu mẫu:
    
    1. 3 teachers
    2. 5 classes
    3. 20 students

## Tiếp theo

- [SELECT & WHERE](select-where.md)
