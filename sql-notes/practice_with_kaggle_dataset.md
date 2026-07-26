## Coffee Taste Test Survey Dataset

### Q1: Top5 Favotate coffee styles for people of different ages

**Answer:**

```sql
WITH favorite_coffee AS(
SELECT age, favorite, COUNT(*) AS num
FROM Coffee
WHERE age != 'NA'
GROUP by age, favorite)

SELECT age, favorite, num
FROM(
  SELECT *, RANK() OVER (
  PARTITION BY age
  ORDER BY num DESC) AS rank
  FROM favorite_coffee) t
WHERE t.rank <= 5
ORDER BY CASE age
	WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
 END, num DESC
```

**Notes:**
- 文本类别要按语义顺序排 → ORDER BY CASE 列 WHEN ... THEN 数字 END(自定义排序)
- Top N 分组排名 → RANK / DENSE_RANK / ROW_NUMBER 三选一,区别在怎么处理并列
