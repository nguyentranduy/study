# Subquery

Subquery (truy vấn con) là một query nằm bên trong một query khác. Subquery có thể được sử dụng trong SELECT, FROM, WHERE, và HAVING.

## Subquery trong WHERE

### Scalar Subquery (trả về 1 giá trị)

```sql
-- Sản phẩm có giá cao hơn giá trung bình
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Đơn hàng gần nhất của khách hàng
SELECT * FROM orders
WHERE order_date = (
    SELECT MAX(order_date) FROM orders WHERE customer_id = 1
);

-- Sản phẩm có giá cao nhất trong category
SELECT * FROM products p1
WHERE price = (
    SELECT MAX(price) FROM products p2 
    WHERE p2.category = p1.category
);
```

### Subquery với IN

```sql
-- Khách hàng đã mua hàng
SELECT * FROM customers
WHERE id IN (SELECT DISTINCT customer_id FROM orders);

-- Sản phẩm chưa được mua
SELECT * FROM products
WHERE id NOT IN (SELECT DISTINCT product_id FROM order_items);

-- Sản phẩm thuộc category có > 10 sản phẩm
SELECT * FROM products
WHERE category IN (
    SELECT category FROM products
    GROUP BY category
    HAVING COUNT(*) > 10
);
```

### Subquery với EXISTS

EXISTS kiểm tra xem subquery có trả về ít nhất 1 row hay không.

```sql
-- Khách hàng có ít nhất 1 đơn hàng
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);

-- Khách hàng chưa có đơn hàng nào
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);

-- Sản phẩm đã được mua trong tháng này
SELECT * FROM products p
WHERE EXISTS (
    SELECT 1 FROM order_items oi
    INNER JOIN orders o ON oi.order_id = o.id
    WHERE oi.product_id = p.id
    AND MONTH(o.order_date) = MONTH(CURDATE())
);
```

### IN vs EXISTS

| IN | EXISTS |
|----|--------|
| Tốt khi subquery trả về ít rows | Tốt khi outer query có ít rows |
| Subquery chạy 1 lần | Subquery chạy cho mỗi row của outer query |
| Không xử lý NULL tốt | Xử lý NULL tốt hơn |

```sql
-- IN: Subquery chạy 1 lần, so sánh với tất cả kết quả
SELECT * FROM products WHERE category IN (SELECT category FROM featured_categories);

-- EXISTS: Subquery chạy cho mỗi product, dừng ngay khi tìm thấy match
SELECT * FROM products p WHERE EXISTS (
    SELECT 1 FROM featured_categories fc WHERE fc.category = p.category
);
```

---

## Subquery trong FROM (Derived Table)

```sql
-- Thống kê khách hàng theo mức chi tiêu
SELECT 
    tier,
    COUNT(*) AS customer_count,
    AVG(total_spent) AS avg_spent
FROM (
    SELECT 
        c.id,
        c.name,
        SUM(o.total) AS total_spent,
        CASE 
            WHEN SUM(o.total) >= 50000000 THEN 'VIP'
            WHEN SUM(o.total) >= 10000000 THEN 'Regular'
            ELSE 'New'
        END AS tier
    FROM customers c
    LEFT JOIN orders o ON c.id = o.customer_id
    GROUP BY c.id, c.name
) customer_stats
GROUP BY tier;

-- Top sản phẩm mỗi category
SELECT * FROM (
    SELECT 
        p.*,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products p
) ranked
WHERE rn <= 3;
```

---

## Subquery trong SELECT

```sql
-- Thêm thông tin tổng hợp vào mỗi row
SELECT 
    p.name,
    p.price,
    p.category,
    (SELECT AVG(price) FROM products WHERE category = p.category) AS category_avg,
    p.price - (SELECT AVG(price) FROM products WHERE category = p.category) AS diff_from_avg
FROM products p;

-- Đếm số đơn hàng của mỗi khách hàng
SELECT 
    c.id,
    c.name,
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) AS order_count,
    (SELECT SUM(total) FROM orders o WHERE o.customer_id = c.id) AS total_spent
FROM customers c;
```

---

## Correlated Subquery

Correlated subquery tham chiếu đến columns của outer query, chạy cho mỗi row của outer query.

```sql
-- Sản phẩm có giá cao nhất trong mỗi category
SELECT * FROM products p1
WHERE price = (
    SELECT MAX(price) FROM products p2 
    WHERE p2.category = p1.category  -- Tham chiếu p1 từ outer query
);

-- Đơn hàng gần nhất của mỗi khách hàng
SELECT * FROM orders o1
WHERE order_date = (
    SELECT MAX(order_date) FROM orders o2
    WHERE o2.customer_id = o1.customer_id
);

-- Sản phẩm có giá cao hơn trung bình của category
SELECT * FROM products p1
WHERE price > (
    SELECT AVG(price) FROM products p2
    WHERE p2.category = p1.category
);
```

---

## Common Table Expression (CTE)

CTE (WITH clause) giúp viết subquery dễ đọc hơn.

### Cú pháp

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

### Ví dụ

```sql
-- Thay vì subquery phức tạp
WITH customer_stats AS (
    SELECT 
        customer_id,
        COUNT(*) AS order_count,
        SUM(total) AS total_spent
    FROM orders
    GROUP BY customer_id
)
SELECT 
    c.name,
    cs.order_count,
    cs.total_spent
FROM customers c
INNER JOIN customer_stats cs ON c.id = cs.customer_id
WHERE cs.total_spent > 10000000;

-- Multiple CTEs
WITH 
monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        SUM(total) AS revenue
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
),
avg_sales AS (
    SELECT AVG(revenue) AS avg_revenue FROM monthly_sales
)
SELECT 
    ms.month,
    ms.revenue,
    a.avg_revenue,
    ms.revenue - a.avg_revenue AS diff
FROM monthly_sales ms
CROSS JOIN avg_sales a
ORDER BY ms.month;
```

### Recursive CTE

```sql
-- Tạo dãy số
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 10
)
SELECT * FROM numbers;

-- Hierarchy (org chart)
WITH RECURSIVE org_chart AS (
    -- Base case: CEO (không có manager)
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: nhân viên có manager
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;
```

---

## Subquery vs JOIN

```sql
-- Subquery
SELECT * FROM products
WHERE category IN (
    SELECT category FROM featured_categories
);

-- JOIN (thường nhanh hơn)
SELECT DISTINCT p.*
FROM products p
INNER JOIN featured_categories fc ON p.category = fc.category;

-- Subquery trong SELECT
SELECT 
    p.name,
    (SELECT c.name FROM categories c WHERE c.id = p.category_id) AS category_name
FROM products p;

-- JOIN (tốt hơn)
SELECT p.name, c.name AS category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id;
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Viết query với subquery:
    
    1. Sản phẩm có giá cao hơn trung bình
    2. Khách hàng có tổng mua cao nhất
    3. Đơn hàng có giá trị cao hơn trung bình của khách hàng đó

!!! example "Bài tập 2"
    Viết query với CTE:
    
    1. Doanh thu theo tháng và so sánh với tháng trước
    2. Top 3 sản phẩm bán chạy mỗi category
    3. Cây phân cấp nhân viên

## Tiếp theo

- [Window Functions](window-functions.md)
