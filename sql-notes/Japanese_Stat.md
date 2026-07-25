## 日本国税調査　

**Q** 1995から2020年まで女性就業者数増長が一番多い産業ランキング

**A1**

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

**A2**

```sql
SELECT col_4, 
	MAX(CASE WHEN time_code = 2020000000 THEN value END) - MAX(CASE WHEN time_code = 1995000000 THEN value END) AS growth
FROM employment
WHERE tab_code = 334 AND cat02_code = 120 
GROUP BY col_4
ORDER BY growth DESC
```

**When to use** 把不同年份/类别横向拉平来对比

**Note**
FIRST_VALUE vs 条件聚合两种写法
GROUP BY 不放裸列(不要SELECT time_code)
