## 日本国税調査　

### Q1:1995から2020年まで女性就業者数増長が一番多い産業ランキング

**A1:** FIRST_VALUE

```sql
WITH all_year AS(
SELECT cat01_code, col_4, time_code, value, value - FIRST_VALUE(value) OVER(
  PARTITION BY cat01_code
  ORDER BY time_code) AS growth
FROM employment
WHERE tab_code = 334 AND cat02_code = 120 
)

SELECT * FROM all_year
WHERE time_code =2020000000
ORDER BY growth DESC
```

**A2:** 条件聚合

```sql
SELECT col_4, 
	MAX(CASE WHEN time_code = 2020000000 THEN value END) - MAX(CASE WHEN time_code = 1995000000 THEN value END) AS growth
FROM employment
WHERE tab_code = 334 AND cat02_code = 120 
GROUP BY col_4
ORDER BY growth DESC
```

**When to use:** 把不同年份/类别横向拉平来对比

**Note:** 
FIRST_VALUE vs 条件聚合两种写法
GROUP BY 不放裸列(不要SELECT time_code)

### Q2:2020年の各産業の従業者数の女性/男性比率を算出

**A:** 

```sql
CREATE table employment AS
SELECT cat01_code AS industry_code, 
  col_4 AS industry, 
  cat02_code AS gender_code, 
  col_6 AS gender, 
  time_code/1000000 AS year, 
  value
FROM  raw
WHERE tab_code = 334;

SELECT industry, 
ROUND(MAX(CASE WHEN year=2020 AND gender='女' THEN value END)*1.0 
      / MAX(CASE WHEN year=2020 AND gender='男' THEN value END),2) AS fmratio
FROM employment
GROUP BY industry
ORDER BY fmratio
```

**Note:**
- 不同类别的值要横向相除/相减 → 先 MAX(CASE WHEN...) 收拢到一行,再运算
- 整数相除会截断 → 乘 1.0 或 CAST
- CREATE TABLE 新表 AS SELECT... → 把清洗/派生结果固化成新表;DROP TABLE IF EXISTS 打头可反复重建


