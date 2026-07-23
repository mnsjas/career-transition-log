# Window Functions — Pattern Notes

Test data: `orders (order_id, user_id, order_date, category, amount)` — 10 rows, sqliteonline.

```sql
CREATE TABLE orders (
  order_id INT, user_id INT, order_date DATE, category TEXT, amount INT
);
INSERT INTO orders (order_id, user_id, order_date, category, amount) VALUES
(1, 101, '2026-01-05', 'book',     1200),
(2, 101, '2026-01-20', 'food',      800),
(3, 101, '2026-03-02', 'book',     1500),
(4, 102, '2026-01-08', 'food',     3000),
(5, 102, '2026-01-08', 'book',     3000),
(6, 102, '2026-02-14', 'gadget',   9800),
(7, 103, '2026-02-01', 'book',      600),
(8, 103, '2026-02-25', 'food',     1200),
(9, 103, '2026-03-15', 'gadget',   4500),
(10, 104, '2026-03-20', 'food',     700);
```
---

## ROW_NUMBER

**Q:** 每个用户的首单是什么时候、买了什么?

```sql
WITH ranked_orders AS (
SELECT *, 
ROW_NUMBER() OVER(
  PARTITION BY user_id
  ORDER BY order_date) AS rn
FROM orders
  )
SELECT * FROM ranked_orders
WHERE rn = 1
```

**Q:** 每个用户的花钱最多的单是什么时候、买了什么?

```sql
WITH ranked_orders AS (
  SELECT *, 
   ROW_NUMBER() OVER (
    PARTITION BY user_id
    ORDER BY amount DESC) AS rn
  FROM orders)
 SELECT * FROM ranked_orders
 WHERE rn = 1
```

**When to use:** 分组内取第 N 条(首单、最新一条、每类目 Top1)。面试高频。

---

## RANK / DENSE_RANK

**Q:** 每个分类中单价最高的交易是哪些？

```sql
WITH ranked_orders AS(
  SELECT *, RANK() OVER (
    PARTITION BY category
    ORDER BY amount DESC) AS rank
  FROM orders
  )
 SELECT * FROM ranked_orders
 WHERE rank = 1
```

**Q:** 所有交易中金额Top5的交易是哪些？

```sql
WITH top5 AS(
  SELECT *, RANK() OVER (
    ORDER BY amount DESC) AS rank
  FROM orders
  )
SELECT * FROM top5
WHERE rank < 6
```

**When to use:** 分组内第N条，最高，topN

---

## LAG

**Q:** 算出每个店铺销售额的前月比

```sql
with ordered_sales As(
SELECT shop_name, year, month, sales, LAG(sales) OVER(
  PARTITION BY shop_name
  order by year, month) AS sales_lag
FROM sales
  )
SELECT *, ROUND(sales*1.0/sales_lag,2) AS sales_ratio
FROM ordered_sales
```

**When to use:** 前月比・前年同月比。

---

## LEAD

**Q:** 算出所有店铺2025年每个月销售额的前年同月的增长率(这个例子不是很好)

```sql
with ordered_sales As(
SELECT shop_name, year, month, sales, LEAD(sales) OVER(
  PARTITION BY shop_name, month
  order by year) AS sales_lead
FROM sales
  )
SELECT *, ROUND((sales_lead/(sales*1.0)-1), 2) AS sales_ratio
FROM ordered_sales
```

**Q:** 算出用户第一次购买后过了多久之后会下第二单

```SQL
WITH new_order AS (
SELECT *, LEAD(order_date) OVER(
  PARTITION BY user_id
  order BY order_date, order_id) AS next_od,
  RANK() OVER (
    PARTITION BY user_id
    order BY order_date, order_id) AS rank
FROM orders
  )
SELECT user_id, order_date, next_od, julianday(next_od) - julianday(order_date) AS time_span
FROM new_order
WHERE rank = 1
```

**When to use:** 复购率，购买间隔，股票价格变化

---

## 移動平均

**Q:**

```sql

```

**When to use:**
