# Analysis with Coffee Taste Test Survey Dataset

## Analysis 1: Caffeine intake (equivalent_cups) by age group

**Data cleaning:**
```sql
SELECT caffeine, COUNT(*) FROM Coffee GROUP BY caffeine;
SELECT cups, COUNT(*) FROM Coffee GROUP BY cups;
drop view if exists coffee_cleaned;

create VIew coffee_cleaned AS
with cleaned AS (
SELECT
  age,
  cups, 
  case cups 
  	when 'Less than 1' then 0
    when 'More than 4' then 5
    ELSE CAST(cups as real)
    end as cleaned_cups,
  caffeine, 
  case caffeine
    when 'Full caffeine' then 1 
    when 'Half caff' then 0.5
    when 'Decaf' then 0
    end as caffeine_level
FRom Coffee
WHEre caffeine <>'NA' AND cups <> 'NA')

SELECT *, 
	caffeine_level * cleaned_cups AS equivalent_cups,
	ROUND(caffeine_level * cleaned_cups,0) AS equivalent_cups_group
from cleaned;
```

**Notes:**
- More than 4' is approximated as 5 and 'Less than 1' as 0 for quantification.

### Q1: Caffeine intake (equivalent_cups) and percentage of people

**Answer:**

```sql
SELECT equivalent_cups_group, 
	   ROUND(count(*)*1.0/SUM(COUNT(*)) OVER (), 2)  as percentage
From coffee_cleaned
group by equivalent_cups_group
order by equivalent_cups_group
```

| equivalent_cups_group | percentage |
|-----------------------:|-----------:|
| 0 | 0.10 |
| 1 | 0.33 |
| 2 | 0.41 |
| 3 | 0.12 |
| 4 | 0.03 |
| 5 | 0.02 |

**Findings:**
- Most people consume 1 or 2 equivalent cups per day.

### Q2: Average Caffeine intake(equivalent_cups) by age group

**Answer:**

```sql
SELECT age,
  SUM(equivalent_cups) as total_cups,
  count(*) AS total_number,
  ROUND(SUM(equivalent_cups)*1.0/count(*), 2) AS average
FROM coffee_cleaned
GROUP BY age
ORDER BY case age
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

**Result:**
| age              | total_cups | total_number | average |
|------------------|-----------:|-------------:|--------:|
| <18 years old    | 18         | 17           | 1.06    |
| 18–24 years old  | 627        | 434          | 1.44    |
| 25–34 years old  | 3077       | 1923         | 1.60    |
| 35–44 years old  | 1656       | 935          | 1.77    |
| 45–54 years old  | 550.5      | 293          | 1.88    |
| 55–64 years old  | 379        | 180          | 2.11    |
| >65 years old    | 189.5      | 92           | 2.06    |

**Findings:**
- caffeine intake tends to increase with age, peaking aroud 55–64 years old before slightly decreasing.
- The average rises from 1.06 cups for under-18s to 2.11 for the 55–64 group.

### Q3: For more details - Caffeine intake(equivalent_cups) and percentage by age group

**Answer:**

```sql
SELECT age, equivalent_cups_group,
  ROUND(COUNT(*) * 1.0 / SUM(COUNT(*)) OVER (PARTITION BY age), 2) AS percentage
FROM coffee_cleaned
GROUP BY age, equivalent_cups_group
ORDER BY case age
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

**Answer(before):**

```sql
SELECT t.age, equivalent_cups_group, s.num_sub, t.num_total, 
ROUND(s.num_sub*1.0/t.num_total,2) AS percentage
FROM(
	SELECT age, COUNT(*) AS num_total
	FROM coffee_adjusted 
	GROUP BY age) t
INNER join (
  SELECT age, equivalent_cups_group, COUNT(*) AS num_sub
  FROM coffee_adjusted
  GROUP BY age,equivalent_cups_group) s
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

**Notes:**
- 「CASE 映射前,先 SELECT DISTINCT 确认所有值都被覆盖,否则漏的变NULL被悄悄丢掉」
- 「算组内占比 → SUM(COUNT(*)) OVER (PARTITION BY 组),比 JOIN 子查询简洁」
- 「JOIN 类型传递意图,同源等价时用 INNER/LEFT,别用 RIGHT 让人困惑」

# Favorite coffee styles
## Q1: Top3 Favotate coffee styles by age group

**Answer1:**

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
WHERE t.rank <= 3
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

**Answer2:**

```sql
WITH favorite_coffee AS(
	SELECT age, favorite, COUNT(*) AS num
	FROM Coffee
	WHERE age != 'NA'
	GROUP by age, favorite),

top3 AS(
  SELECT *, ROW_NUMBER() OVER (
  PARTITION BY age
  ORDER BY num DESC) AS rank
  FROM favorite_coffee)

SELECT age,  
MAX(CASE when rank = 1 then favorite END) as top1,
MAX(case when rank = 2 THEN favorite END) AS top2,
MAX(case when rank = 3 THEN favorite END) AS top3
FROM top3
group by age
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

**Result**
| age              | top1                 | top2      | top3                 |
|------------------|----------------------|-----------|----------------------|
| <18 years old    | Latte                | Pourover  | Regular drip coffee  |
| 18-24 years old  | Pourover             | Latte     | Cortado              |
| 25-34 years old  | Pourover             | Latte     | Regular drip coffee  |
| 35-44 years old  | Pourover             | Latte     | Regular drip coffee  |
| 45-54 years old  | Pourover             | Latte     | Regular drip coffee  |
| 55-64 years old  | Regular drip coffee  | Pourover  | Latte                |
| >65 years old    | Regular drip coffee  | Pourover  | Latte                |

**Finds**
- Preference towards Latte decreases with age
- ラテに対する好みは年の増加とともに減ります

**Notes:**
- 文本类别要按语义顺序排 → ORDER BY CASE 列 WHEN ... THEN 数字 END(自定义排序)
- Top N 分组排名 → RANK / DENSE_RANK / ROW_NUMBER 三选一,区别在怎么处理并列

### Q3: Coffee preference (Favorite coffee styles, Roast level, Flavor, with or without suger) by races

**Answer1:**
```sql
SELECT DISTINCT ethnicity_race 
FROM  Coffee

DROP TABLE IF EXISTS coffee_preferences;
CREATE table coffee_preferences AS
  SELECT favorite, sweetener, style, roast_level, ethnicity_race
  FROM Coffee
  WHERE ethnicity_race <>'NA';

SELECT * 
FROM coffee_preferences 
LIMIT 20

with favorite_count AS (
  SELECT ethnicity_race, favorite, COUNT(*) AS num
  from coffee_preferences
  group by ethnicity_race, favorite),

favorite_rank AS (
SELECT 
	ethnicity_race, 
    favorite, 
	ROW_NUMBER() OVER(
      PARTITION BY ethnicity_race
      ORDER BY num DESC) as rank
FROM favorite_count)

SELECT 
	r1.ethnicity_race, 
	r1.favorite AS first, 
    r2.favorite as second, 
    r3.favorite as third
FROM favorite_rank r1
inner join favorite_rank r2
inner join favorite_rank r3
ON r1.ethnicity_race = r2.ethnicity_race
AND r2.ethnicity_race = r3.ethnicity_race
WHERE r1.rank = 1 
AND r2.rank = 2
and r3.rank = 3
```

**Answer2:**

```sql
with favorite_count AS (
  SELECT ethnicity_race, favorite, COUNT(*) AS num
  from coffee_preferences
  group by ethnicity_race, favorite),

favorite_rank AS (
SELECT 
	ethnicity_race, 
    favorite, 
	ROW_NUMBER() OVER(
      PARTITION BY ethnicity_race
      ORDER BY num DESC) as rank
FROM favorite_count)

SELECT ethnicity_race,
	MAX(case when rank = 1 then favorite end) as first,
    MAX(case when rank = 2 then favorite end) as second,
	MAX(case when rank = 3 then favorite end) as third
FROM favorite_rank
group by ethnicity_race
```

**Notes:**
- CREATE TABLE の文末に ; を付けて、次の SELECT を別のSQL文として実行します
- GROUP BY の不足に気を付けろ
- 同じSELECT句で付けた別名を参照できないので、CTEを使おう
- SQLの実行順序はFROM、WHERE、GROUP BY、集約関数、SELECT、ウィンドウ関数（ROW_NUMBER() や RANK()）、ORDER BY
- 取组内Top-N并排成列:自连接会在凑不齐N个时静默丢整组;条件聚合 MAX(CASE WHEN rank=n) 会保留行、缺的显示NULL —— 后者更安全
