# Index & Performance

Index là cấu trúc dữ liệu giúp tăng tốc độ truy vấn bằng cách tạo "mục lục" cho bảng.

## Index là gì?

### Không có Index

```
Query: SELECT * FROM users WHERE email = 'test@example.com'

Cách hoạt động: Full Table Scan
- Duyệt qua TẤT CẢ rows trong bảng
- Kiểm tra từng row xem email có khớp không
- Với 1 triệu rows → kiểm tra 1 triệu lần
```

### Có Index

```
Query: SELECT * FROM users WHERE email = 'test@example.com'

Cách hoạt động: Index Seek
- Tìm trong B-Tree index
- Nhảy trực tiếp đến row cần tìm
- Với 1 triệu rows → chỉ cần ~20 bước (log2(1M))
```

---

## Các loại Index

### B-Tree Index (mặc định)

Phù hợp cho: `=`, `>`, `<`, `>=`, `<=`, `BETWEEN`, `LIKE 'abc%'`

```sql
-- Tạo index
CREATE INDEX idx_users_email ON users(email);

-- Xóa index
DROP INDEX idx_users_email ON users;
```

### Hash Index

Chỉ phù hợp cho `=` (equality). Nhanh hơn B-Tree cho equality nhưng không hỗ trợ range.

```sql
-- MySQL (chỉ MEMORY engine)
CREATE INDEX idx_users_email ON users(email) USING HASH;
```

### Full-Text Index

Cho tìm kiếm văn bản.

```sql
CREATE FULLTEXT INDEX idx_products_name ON products(name, description);

-- Sử dụng
SELECT * FROM products
WHERE MATCH(name, description) AGAINST('laptop gaming');
```

### Composite Index (Multi-column)

Index trên nhiều columns.

```sql
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```

---

## Composite Index và thứ tự columns

### Quy tắc Leftmost Prefix

Composite index `(A, B, C)` có thể được sử dụng cho:

- Query trên `A`
- Query trên `A, B`
- Query trên `A, B, C`

**KHÔNG** được sử dụng cho:

- Query chỉ trên `B`
- Query chỉ trên `C`
- Query trên `B, C`

```sql
-- Index: (customer_id, order_date, status)
CREATE INDEX idx_orders_composite ON orders(customer_id, order_date, status);

-- SỬ DỤNG được index
SELECT * FROM orders WHERE customer_id = 1;
SELECT * FROM orders WHERE customer_id = 1 AND order_date = '2024-01-15';
SELECT * FROM orders WHERE customer_id = 1 AND order_date > '2024-01-01' AND status = 'completed';

-- KHÔNG sử dụng được index (cần tạo index riêng)
SELECT * FROM orders WHERE order_date = '2024-01-15';  -- Thiếu customer_id
SELECT * FROM orders WHERE status = 'completed';       -- Thiếu customer_id, order_date
```

!!! warning "Lưu ý với PostgreSQL"
    PostgreSQL **không** tự động sử dụng composite index `(A, B)` cho queries chỉ filter trên column `B`. Bạn cần tạo index riêng cho column `B` nếu có queries filter riêng trên `B`.

### Thứ tự columns trong Composite Index

Đặt column có **selectivity cao** (nhiều giá trị unique) trước.

```sql
-- BAD: status chỉ có vài giá trị (pending, completed, cancelled)
CREATE INDEX idx_bad ON orders(status, customer_id);

-- GOOD: customer_id có nhiều giá trị unique hơn
CREATE INDEX idx_good ON orders(customer_id, status);
```

---

## EXPLAIN - Phân tích Query

### MySQL EXPLAIN

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 1;

-- Kết quả
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | orders | ref  | idx_customer  | idx_customer | 4 | const | 15 | Using where |
+----+-------------+--------+------+---------------+------+---------+------+------+-------------+
```

### Các giá trị type (từ tốt đến xấu)

| Type | Mô tả | Performance |
|------|-------|-------------|
| `system` | Bảng chỉ có 1 row | Tốt nhất |
| `const` | Tìm bằng PRIMARY KEY hoặc UNIQUE | Rất tốt |
| `eq_ref` | JOIN với PRIMARY KEY | Rất tốt |
| `ref` | Sử dụng index, có thể nhiều rows | Tốt |
| `range` | Index range scan (BETWEEN, >, <) | Khá tốt |
| `index` | Full index scan | Trung bình |
| `ALL` | Full table scan | Xấu |

### EXPLAIN ANALYZE (MySQL 8.0+)

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 1;

-- Hiển thị thời gian thực tế
-> Index lookup on orders using idx_customer (customer_id=1)  
   (cost=2.10 rows=15) (actual time=0.025..0.035 rows=15 loops=1)
```

---

## Khi nào nên tạo Index

### NÊN tạo Index

```sql
-- 1. Columns trong WHERE thường xuyên
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_status ON orders(status);

-- 2. Columns trong JOIN
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- 3. Columns trong ORDER BY
CREATE INDEX idx_products_price ON products(price);

-- 4. Columns trong GROUP BY
CREATE INDEX idx_orders_date ON orders(order_date);

-- 5. Foreign Keys
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

### KHÔNG NÊN tạo Index

```sql
-- 1. Bảng nhỏ (< 1000 rows)
-- Full scan có thể nhanh hơn index lookup

-- 2. Columns có ít giá trị unique (low cardinality)
-- VD: gender (M/F), status (active/inactive)

-- 3. Columns thường xuyên UPDATE
-- Mỗi UPDATE phải cập nhật cả index

-- 4. Columns hiếm khi dùng trong WHERE/JOIN/ORDER BY
```

---

## Covering Index

Index chứa tất cả columns cần thiết cho query → không cần đọc table.

```sql
-- Query
SELECT customer_id, order_date, total FROM orders WHERE customer_id = 1;

-- Covering index
CREATE INDEX idx_covering ON orders(customer_id, order_date, total);

-- EXPLAIN sẽ hiển thị "Using index" trong Extra
```

---

## Index và NULL

```sql
-- B-Tree index CÓ THỂ index NULL values
CREATE INDEX idx_users_phone ON users(phone);

-- Query này SỬ DỤNG được index
SELECT * FROM users WHERE phone IS NULL;

-- Nhưng NULL không được index trong UNIQUE constraint
-- Có thể có nhiều rows với NULL
```

---

## Query Optimization Tips

### 1. Tránh SELECT *

```sql
-- BAD
SELECT * FROM orders WHERE customer_id = 1;

-- GOOD: Chỉ lấy columns cần thiết
SELECT id, order_date, total FROM orders WHERE customer_id = 1;
```

### 2. Tránh functions trên indexed columns

```sql
-- BAD: Index không được sử dụng
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- GOOD: Index được sử dụng
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

### 3. Tránh LIKE với wildcard đầu

```sql
-- BAD: Full table scan
SELECT * FROM products WHERE name LIKE '%laptop%';

-- GOOD: Sử dụng được index
SELECT * FROM products WHERE name LIKE 'laptop%';

-- Nếu cần search anywhere, dùng Full-Text Index
```

### 4. Sử dụng LIMIT

```sql
-- BAD: Lấy tất cả rồi mới limit ở application
SELECT * FROM products ORDER BY created_at DESC;

-- GOOD: Limit ở database
SELECT * FROM products ORDER BY created_at DESC LIMIT 10;
```

### 5. Tránh OR, dùng UNION hoặc IN

```sql
-- BAD: Có thể không dùng index
SELECT * FROM products WHERE category = 'A' OR category = 'B';

-- GOOD: Dùng IN
SELECT * FROM products WHERE category IN ('A', 'B');

-- Hoặc UNION (nếu cần)
SELECT * FROM products WHERE category = 'A'
UNION ALL
SELECT * FROM products WHERE category = 'B';
```

### 6. Batch operations

```sql
-- BAD: Nhiều queries
INSERT INTO logs VALUES (1, 'log1');
INSERT INTO logs VALUES (2, 'log2');
INSERT INTO logs VALUES (3, 'log3');

-- GOOD: Batch insert
INSERT INTO logs VALUES (1, 'log1'), (2, 'log2'), (3, 'log3');
```

---

## Monitoring và Maintenance

### Xem index usage (MySQL)

```sql
-- Xem tất cả indexes của table
SHOW INDEX FROM orders;

-- Xem index statistics
SELECT * FROM sys.schema_index_statistics 
WHERE table_name = 'orders';

-- Tìm unused indexes
SELECT * FROM sys.schema_unused_indexes;
```

### Rebuild Index

```sql
-- MySQL
ALTER TABLE orders ENGINE=InnoDB;  -- Rebuild tất cả indexes
OPTIMIZE TABLE orders;

-- PostgreSQL
REINDEX TABLE orders;
```

### Analyze Table

```sql
-- Cập nhật statistics cho query optimizer
ANALYZE TABLE orders;  -- MySQL
ANALYZE orders;        -- PostgreSQL
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Phân tích và tối ưu:
    
    1. Sử dụng EXPLAIN để phân tích query chậm
    2. Tạo index phù hợp
    3. So sánh performance trước và sau

!!! example "Bài tập 2"
    Thiết kế index cho các queries:
    
    1. Tìm đơn hàng theo customer và ngày
    2. Thống kê doanh thu theo tháng
    3. Tìm sản phẩm theo category và giá

## Tiếp theo

- [Thực hành SQL](../practice.md)
