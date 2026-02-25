# Window Functions

Window Functions cho phép thực hiện tính toán trên một tập hợp các rows liên quan đến row hiện tại, mà không cần GROUP BY.

## Tổng quan

### Window Function vs Aggregate Function

| Aggregate Function | Window Function |
|-------------------|-----------------|
| Gộp nhiều rows thành 1 row | Giữ nguyên số rows |
| Cần GROUP BY | Không cần GROUP BY |
| `SUM(price)` | `SUM(price) OVER (...)` |

```sql
-- Aggregate: 1 row kết quả
SELECT category, SUM(price) AS total
FROM products
GROUP BY category;

-- Window: Giữ nguyên tất cả rows, thêm cột tổng
SELECT 
    name,
    category,
    price,
    SUM(price) OVER (PARTITION BY category) AS category_total
FROM products;
```

---

## Cú pháp

```sql
function_name() OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column3, column4, ...]
    [frame_clause]
)
```

- **PARTITION BY**: Chia dữ liệu thành các nhóm (như GROUP BY nhưng không gộp)
- **ORDER BY**: Sắp xếp rows trong mỗi partition
- **Frame clause**: Xác định phạm vi rows để tính toán

---

## Ranking Functions

### ROW_NUMBER()

Đánh số thứ tự liên tục, không có số trùng.

```sql
SELECT 
    name,
    category,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS overall_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS category_rank
FROM products;

-- Kết quả
+------------+-------------+----------+--------------+---------------+
|    name    |  category   |  price   | overall_rank | category_rank |
+------------+-------------+----------+--------------+---------------+
| iPhone 15  | Phones      | 25000000 |            1 |             1 |
| Laptop Pro | Electronics | 22000000 |            2 |             1 |
| Samsung S24| Phones      | 20000000 |            3 |             2 |
| Monitor    | Electronics | 8000000  |            4 |             2 |
+------------+-------------+----------+--------------+---------------+
```

### RANK()

Đánh số thứ tự, có số trùng khi giá trị bằng nhau, bỏ qua số tiếp theo.

```sql
SELECT 
    name,
    price,
    RANK() OVER (ORDER BY price DESC) AS rank
FROM products;

-- Kết quả (nếu có 2 sản phẩm cùng giá)
+------------+----------+------+
|    name    |  price   | rank |
+------------+----------+------+
| Product A  | 25000000 |    1 |
| Product B  | 25000000 |    1 |  -- Cùng rank
| Product C  | 20000000 |    3 |  -- Bỏ qua rank 2
+------------+----------+------+
```

### DENSE_RANK()

Như RANK() nhưng không bỏ qua số.

```sql
SELECT 
    name,
    price,
    DENSE_RANK() OVER (ORDER BY price DESC) AS dense_rank
FROM products;

-- Kết quả
+------------+----------+------------+
|    name    |  price   | dense_rank |
+------------+----------+------------+
| Product A  | 25000000 |          1 |
| Product B  | 25000000 |          1 |  -- Cùng rank
| Product C  | 20000000 |          2 |  -- Tiếp tục với 2
+------------+----------+------------+
```

### NTILE(n)

Chia dữ liệu thành n nhóm bằng nhau.

```sql
-- Chia sản phẩm thành 4 nhóm theo giá
SELECT 
    name,
    price,
    NTILE(4) OVER (ORDER BY price DESC) AS quartile
FROM products;

-- Kết quả
+------------+----------+----------+
|    name    |  price   | quartile |
+------------+----------+----------+
| Product A  | 25000000 |        1 |  -- Top 25%
| Product B  | 22000000 |        1 |
| Product C  | 20000000 |        2 |  -- 25-50%
| Product D  | 18000000 |        2 |
| Product E  | 15000000 |        3 |  -- 50-75%
| Product F  | 10000000 |        3 |
| Product G  | 5000000  |        4 |  -- Bottom 25%
| Product H  | 3000000  |        4 |
+------------+----------+----------+
```

---

## Aggregate Window Functions

```sql
SELECT 
    name,
    category,
    price,
    -- Tổng theo category
    SUM(price) OVER (PARTITION BY category) AS category_total,
    -- Trung bình theo category
    AVG(price) OVER (PARTITION BY category) AS category_avg,
    -- Số sản phẩm trong category
    COUNT(*) OVER (PARTITION BY category) AS category_count,
    -- Min/Max trong category
    MIN(price) OVER (PARTITION BY category) AS category_min,
    MAX(price) OVER (PARTITION BY category) AS category_max
FROM products;
```

### Running Total (Tổng tích lũy)

```sql
SELECT 
    order_date,
    total,
    SUM(total) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Kết quả
+------------+---------+---------------+
| order_date |  total  | running_total |
+------------+---------+---------------+
| 2024-01-01 |  500000 |        500000 |
| 2024-01-02 |  750000 |       1250000 |
| 2024-01-03 |  300000 |       1550000 |
| 2024-01-04 | 1000000 |       2550000 |
+------------+---------+---------------+
```

### Running Total theo partition

```sql
SELECT 
    customer_id,
    order_date,
    total,
    SUM(total) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS customer_running_total
FROM orders;
```

---

## Value Functions

### LAG() và LEAD()

Lấy giá trị từ row trước/sau.

```sql
SELECT 
    order_date,
    total,
    LAG(total) OVER (ORDER BY order_date) AS prev_total,
    LEAD(total) OVER (ORDER BY order_date) AS next_total,
    total - LAG(total) OVER (ORDER BY order_date) AS diff_from_prev
FROM orders;

-- Kết quả
+------------+---------+------------+------------+----------------+
| order_date |  total  | prev_total | next_total | diff_from_prev |
+------------+---------+------------+------------+----------------+
| 2024-01-01 |  500000 |       NULL |     750000 |           NULL |
| 2024-01-02 |  750000 |     500000 |     300000 |         250000 |
| 2024-01-03 |  300000 |     750000 |    1000000 |        -450000 |
| 2024-01-04 | 1000000 |     300000 |       NULL |         700000 |
+------------+---------+------------+------------+----------------+
```

### LAG/LEAD với offset và default

```sql
SELECT 
    month,
    revenue,
    LAG(revenue, 1, 0) OVER (ORDER BY month) AS prev_month,
    LAG(revenue, 12, 0) OVER (ORDER BY month) AS same_month_last_year
FROM monthly_sales;
```

### FIRST_VALUE() và LAST_VALUE()

```sql
SELECT 
    name,
    category,
    price,
    FIRST_VALUE(name) OVER (
        PARTITION BY category 
        ORDER BY price DESC
    ) AS most_expensive,
    LAST_VALUE(name) OVER (
        PARTITION BY category 
        ORDER BY price DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS cheapest
FROM products;
```

### NTH_VALUE()

```sql
-- Lấy sản phẩm đắt thứ 2 trong mỗi category
SELECT 
    name,
    category,
    price,
    NTH_VALUE(name, 2) OVER (
        PARTITION BY category 
        ORDER BY price DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_most_expensive
FROM products;
```

---

## Frame Clause

Frame clause xác định phạm vi rows để tính toán.

### Cú pháp

```sql
ROWS BETWEEN start AND end
RANGE BETWEEN start AND end
```

### Các giá trị start/end

| Giá trị | Mô tả |
|---------|-------|
| `UNBOUNDED PRECEDING` | Từ đầu partition |
| `n PRECEDING` | n rows trước |
| `CURRENT ROW` | Row hiện tại |
| `n FOLLOWING` | n rows sau |
| `UNBOUNDED FOLLOWING` | Đến cuối partition |

### Ví dụ

```sql
-- Moving average 3 ngày
SELECT 
    order_date,
    total,
    AVG(total) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3days
FROM orders;

-- Moving average 7 ngày (3 trước + hiện tại + 3 sau)
SELECT 
    order_date,
    total,
    AVG(total) OVER (
        ORDER BY order_date
        ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
    ) AS moving_avg_7days
FROM orders;

-- Tổng từ đầu đến hiện tại (mặc định)
SUM(total) OVER (ORDER BY order_date)
-- Tương đương với
SUM(total) OVER (
    ORDER BY order_date 
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

---

## Ứng dụng thực tế

### Top N mỗi nhóm

```sql
-- Top 3 sản phẩm bán chạy mỗi category
WITH ranked AS (
    SELECT 
        p.name,
        p.category,
        SUM(oi.quantity) AS total_sold,
        ROW_NUMBER() OVER (
            PARTITION BY p.category 
            ORDER BY SUM(oi.quantity) DESC
        ) AS rn
    FROM products p
    INNER JOIN order_items oi ON p.id = oi.product_id
    GROUP BY p.id, p.name, p.category
)
SELECT * FROM ranked WHERE rn <= 3;
```

### So sánh với tháng trước

```sql
WITH monthly AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        SUM(total) AS revenue
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS diff,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) / 
          LAG(revenue) OVER (ORDER BY month) * 100, 2) AS growth_percent
FROM monthly;
```

### Phân tích RFM

```sql
WITH customer_rfm AS (
    SELECT 
        customer_id,
        DATEDIFF(CURDATE(), MAX(order_date)) AS recency,
        COUNT(*) AS frequency,
        SUM(total) AS monetary
    FROM orders
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT 
        customer_id,
        recency,
        frequency,
        monetary,
        NTILE(5) OVER (ORDER BY recency DESC) AS r_score,
        NTILE(5) OVER (ORDER BY frequency) AS f_score,
        NTILE(5) OVER (ORDER BY monetary) AS m_score
    FROM customer_rfm
)
SELECT 
    customer_id,
    CONCAT(r_score, f_score, m_score) AS rfm_segment,
    CASE 
        WHEN r_score >= 4 AND f_score >= 4 THEN 'Champions'
        WHEN r_score >= 3 AND f_score >= 3 THEN 'Loyal'
        WHEN r_score >= 4 AND f_score <= 2 THEN 'New Customers'
        WHEN r_score <= 2 AND f_score >= 3 THEN 'At Risk'
        ELSE 'Others'
    END AS customer_segment
FROM rfm_scores;
```

---

## Bài tập thực hành

!!! example "Bài tập 1"
    Viết query với Window Functions:
    
    1. Xếp hạng sản phẩm theo doanh thu trong mỗi category
    2. Tính running total doanh thu theo ngày
    3. So sánh doanh thu mỗi tháng với tháng trước

!!! example "Bài tập 2"
    Phân tích nâng cao:
    
    1. Moving average 7 ngày của doanh thu
    2. Top 5 khách hàng mỗi tháng
    3. Tỷ lệ đóng góp của mỗi sản phẩm trong category

## Tiếp theo

- [Index & Performance](index-performance.md)
