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

**Q:**

```sql

```

**When to use:**

---

## LAG

**Q:**

```sql

```

**When to use:** 前月比・前年同月比。

---

## LEAD

**Q:**

```sql

```

**When to use:**

---

## 移動平均

**Q:**

```sql

```

**When to use:**
