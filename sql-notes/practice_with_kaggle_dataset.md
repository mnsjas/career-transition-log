## Coffee Taste Test Survey Dataset

### Q1: Top5 Favotate coffee styles by age group

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

### Q2: Caffeine intake(equivalent_cups) and percentage by age group

**Answer:**

```sql
WITH coffee_adjusted AS(
	SELECT age, cups, caffeine, (cups * caffeine_level) AS equivalent_cups
	FROM(
		SELECT age, cups, caffeine, 
		CASE caffeine 
			WHEN 'Full caffeine' THEN 1
    		WHEN 'Half cafe' THEN 0.5
    		WHEN 'Decaf' THEN 0
    		WHEN 'NA' THEN NULL 
		END AS caffeine_level
    	FROM Coffee) c
	WHERE cups * caffeine_level IS NOT NULL 
  		AND age <>'NA'
	ORDER BY equivalent_cups DESC)
    
SELECT t.age, equivalent_cups, s.num_sub, t.num_total, 
ROUND(s.num_sub*1.0/t.num_total,2) AS percentage
FROM(
	SELECT age, COUNT(*) AS num_total
	FROM coffee_adjusted 
	GROUP BY age) t
INNER join (
  SELECT age, equivalent_cups, COUNT(*) AS num_sub
  FROM coffee_adjusted
  GROUP BY age,equivalent_cups) s
ON t.age = s.age
ORDER BY CASE s.age
	WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
 END
```

**AnswerB(from claude):**

```sql
SELECT age, equivalent_cups,
  COUNT(*) AS num_sub,
  SUM(COUNT(*)) OVER (PARTITION BY age) AS num_total,
  ROUND(COUNT(*) * 1.0 / SUM(COUNT(*)) OVER (PARTITION BY age), 2) AS percentage
FROM coffee_adjusted
GROUP BY age, equivalent_cups
ORDER BY age;
```

**Notes:**
- 「CASE 映射前,先 SELECT DISTINCT 确认所有值都被覆盖,否则漏的变NULL被悄悄丢掉」
- 「算组内占比 → SUM(COUNT(*)) OVER (PARTITION BY 组),比 JOIN 子查询简洁」
- 「JOIN 类型传递意图,同源等价时用 INNER/LEFT,别用 RIGHT 让人困惑」
