# GROUP BY & HAVING

GROUP BY cho phép nhóm các rows có cùng giá trị và áp dụng các hàm tổng hợp (aggregate functions) lên mỗi nhóm.

## GROUP BY cơ bản

### Cú pháp

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

### Ví dụ

```sql
-- Đếm số sản phẩm theo category
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category;

-- Kết quả
+-------------+---------------+
|  category   | product_count |
+-------------+---------------+
| Electronics |            15 |
| Phones      |            10 |
| Accessories |            25 |
+-------------+---------------+

-- Tổng doanh thu theo khách hàng
SELECT customer_id, SUM(total) AS total_spent
FROM orders
GROUP BY customer_id;

-- Số đơn hàng theo tháng
SELECT 
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year, month;
```

---

## GROUP BY với nhiều columns

```sql
-- Thống kê theo category và năm
SELECT 
    p.category,
    YEAR(o.order_date) AS year,
    SUM(oi.quantity) AS total_sold,
    SUM(oi.quantity * oi.price) AS revenue
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.id
INNER JOIN orders o ON oi.order_id = o.id
GROUP BY p.category, YEAR(o.order_date)
ORDER BY p.category, year;

-- Thống kê theo thành phố và trạng thái đơn hàng
SELECT 
    c.city,
    o.status,
    COUNT(*) AS order_count
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
GROUP BY c.city, o.status
ORDER BY c.city, o.status;
```

---

## HAVING - Lọc sau khi GROUP

HAVING dùng để lọc các nhóm sau khi đã GROUP BY. Khác với WHERE (lọc trước khi GROUP).

### Cú pháp

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;
```

### WHERE vs HAVING

| WHERE | HAVING |
|-------|--------|
| Lọc rows **trước** GROUP BY | Lọc groups **sau** GROUP BY |
| Không dùng được aggregate functions | Dùng được aggregate functions |
| Áp dụng cho từng row | Áp dụng cho từng group |

```sql
-- WHERE: Lọc trước khi group
SELECT category, COUNT(*) AS product_count
FROM products
WHERE price > 1000000  -- Chỉ đếm sản phẩm có giá > 1 triệu
GROUP BY category;

-- HAVING: Lọc sau khi group
SELECT category, COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 5;  -- Chỉ lấy category có > 5 sản phẩm

-- Kết hợp WHERE và HAVING
SELECT category, AVG(price) AS avg_price
FROM products
WHERE stock > 0           -- Chỉ tính sản phẩm còn hàng
GROUP BY category
HAVING AVG(price) > 5000000;  -- Chỉ lấy category có giá TB > 5 triệu
```

### Ví dụ HAVING

```sql
-- Khách hàng có tổng mua > 10 triệu
SELECT 
    c.id,
    c.name,
    SUM(o.total) AS total_spent
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
HAVING SUM(o.total) > 10000000
ORDER BY total_spent DESC;

-- Category có doanh thu > 100 triệu trong năm 2024
SELECT 
    p.category,
    SUM(oi.quantity * oi.price) AS revenue
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.id
INNER JOIN orders o ON oi.order_id = o.id
WHERE YEAR(o.order_date) = 2024
GROUP BY p.category
HAVING SUM(oi.quantity * oi.price) > 100000000;

-- Sản phẩm được mua >= 10 lần
SELECT 
    p.id,
    p.name,
    COUNT(*) AS times_ordered,
    SUM(oi.quantity) AS total_quantity
FROM products p
INNER JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name
HAVING COUNT(*) >= 10
ORDER BY times_ordered DESC;
```

---

## Thứ tự thực thi

```sql
SELECT column, aggregate_function()  -- 5. Select
FROM table                           -- 1. From
WHERE condition                      -- 2. Where (lọc rows)
GROUP BY column                      -- 3. Group By
HAVING condition                     -- 4. Having (lọc groups)
ORDER BY column                      -- 6. Order By
LIMIT n;                             -- 7. Limit
```

---

## GROUP BY với ROLLUP

ROLLUP tạo thêm các subtotal và grand total.

```sql
-- MySQL
SELECT 
    COALESCE(category, 'TOTAL') AS category,
    COALESCE(YEAR(created_at), 0) AS year,
    COUNT(*) AS product_count,
    SUM(price) AS total_price
FROM products
GROUP BY category, YEAR(created_at) WITH ROLLUP;

-- Kết quả
+-------------+------+---------------+-------------+
|  category   | year | product_count | total_price |
+-------------+------+---------------+-------------+
| Electronics | 2023 |             5 |    50000000 |
| Electronics | 2024 |            10 |   120000000 |
| Electronics |    0 |            15 |   170000000 |  -- Subtotal Electronics
| Phones      | 2023 |             3 |    45000000 |
| Phones      | 2024 |             7 |    98000000 |
| Phones      |    0 |            10 |   143000000 |  -- Subtotal Phones
| TOTAL       |    0 |            25 |   313000000 |  -- Grand Total
+-------------+------+---------------+-------------+
```

---

## GROUP BY với CASE

```sql
-- Phân loại khách hàng theo mức chi tiêu
SELECT 
    CASE 
        WHEN SUM(total) >= 50000000 THEN 'VIP'
        WHEN SUM(total) >= 10000000 THEN 'Regular'
        ELSE 'New'
    END AS customer_tier,
    COUNT(*) AS customer_count
FROM (
    SELECT customer_id, SUM(total) AS total
    FROM orders
    GROUP BY customer_id
) customer_totals
GROUP BY customer_tier;

-- Thống kê đơn hàng theo khoảng giá
SELECT 
    CASE 
        WHEN total < 500000 THEN 'Under 500K'
        WHEN total < 2000000 THEN '500K - 2M'
        WHEN total < 10000000 THEN '2M - 10M'
        ELSE 'Over 10M'
    END AS price_range,
    COUNT(*) AS order_count,
    SUM(total) AS total_revenue
FROM orders
GROUP BY 
    CASE 
        WHEN total < 500000 THEN 'Under 500K'
        WHEN total < 2000000 THEN '500K - 2M'
        WHEN total < 10000000 THEN '2M - 10M'
        ELSE 'Over 10M'
    END
ORDER BY MIN(total);
```

---

## Các hàm Aggregate phổ biến

```sql
SELECT 
    category,
    COUNT(*) AS total_products,
    COUNT(DISTINCT supplier_id) AS supplier_count,
    SUM(stock) AS total_stock,
    AVG(price) AS avg_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price,
    GROUP_CONCAT(name SEPARATOR ', ') AS product_names  -- MySQL
FROM products
GROUP BY category;
```

### GROUP_CONCAT (MySQL)

```sql
-- Liệt kê tên sản phẩm theo category
SELECT 
    category,
    GROUP_CONCAT(name ORDER BY name SEPARATOR ', ') AS products
FROM products
GROUP BY category;

-- Kết quả
+-------------+----------------------------------+
|  category   |            products              |
+-------------+----------------------------------+
| Electronics | Laptop, Monitor, Keyboard        |
| Phones      | iPhone 15, Samsung S24, Xiaomi   |
+-------------+----------------------------------+
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Viết query thống kê:
    
    1. Số lượng đơn hàng theo trạng thái
    2. Doanh thu theo tháng trong năm 2024
    3. Top 5 sản phẩm bán chạy nhất

!!! example "Bài tập 2"
    Viết query với HAVING:
    
    1. Khách hàng có >= 3 đơn hàng
    2. Category có doanh thu > 50 triệu
    3. Tháng có doanh thu giảm so với tháng trước

## Tiếp theo

- [Subquery](../advanced/subquery.md)
