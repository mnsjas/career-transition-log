# Analysis with Coffee Taste Test Survey Dataset

## Overview
This analysis explores caffeine consumption and coffee preferences using the
Coffee Taste Test Survey dataset (~4,000 responses). It examines patterns by
age group and ethnicity, using SQL (SQLite) with CTEs, window functions, and
conditional aggregation.

**Skills demonstrated:** data cleaning with VIEWs, window functions
(ROW_NUMBER, SUM OVER), conditional aggregation, hypothesis testing.

**Data validation:**
```sql
SELECT caffeine, COUNT(*) FROM Coffee GROUP BY caffeine;
SELECT cups, COUNT(*) FROM Coffee GROUP BY cups;
SELECT additions, COUNT(*) FROM Coffee GROUP BY additions;
```

**Data cleaning:**
```sql
DROP VIEW IF EXISTS coffee_cleaned;

CREATE VIEW coffee_cleaned AS
WITH cleaned AS (
  SELECT
    age,
    cups,
    CASE cups
      WHEN 'Less than 1' THEN 0
      WHEN 'More than 4' THEN 5
      ELSE CAST(cups AS REAL)
    END AS cleaned_cups,
    caffeine,
    CASE caffeine
      WHEN 'Full caffeine' THEN 1
      WHEN 'Half caff' THEN 0.5
      WHEN 'Decaf' THEN 0
    END AS caffeine_level,
    additions,
    CASE
      WHEN additions LIKE '%Flavor syrup%' THEN 'Y'
      WHEN additions LIKE '%Sugar or sweetener%' THEN 'Y'
      ELSE 'N'
    END AS sugar_dummy
  FROM Coffee
  WHERE age <> 'NA'
    AND cups <> 'NA'
    AND caffeine <> 'NA'
    AND additions <> 'NA'
)
SELECT *,
  caffeine_level * cleaned_cups AS equivalent_cups,
  ROUND(caffeine_level * cleaned_cups, 0) AS equivalent_cups_group
FROM cleaned;

SELECT * FROM coffee_cleaned;
```

**Notes:**
- `More than 4` is approximated as `5` and `Less than 1` as `0` for quantification.
- Confirmed from the validation step that `caffeine` and `cups` contain no valid values outside the CASE mapping.

## Analysis 1: Caffeine intake (equivalent cups) by age group

### Q1: Caffeine intake (equivalent cups) and percentage of people

**Answer:**
```sql
SELECT equivalent_cups_group,
       ROUND(COUNT(*) * 1.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM coffee_cleaned
GROUP BY equivalent_cups_group
ORDER BY equivalent_cups_group;
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

### Q2: Average caffeine intake (equivalent cups) by age group

**Answer:**
```sql
SELECT age,
       SUM(equivalent_cups) AS total_cups,
       COUNT(*) AS total_number,
       ROUND(SUM(equivalent_cups) * 1.0 / COUNT(*), 2) AS average
FROM coffee_cleaned
GROUP BY age
ORDER BY CASE age
    WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
  END;
```

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
- Caffeine intake tends to increase with age, peaking around 55–64 years old before slightly decreasing.
- The average rises from 1.06 cups for under-18s to 2.11 for the 55–64 group.

### Q3: For more detail — caffeine intake (equivalent cups) and percentage by age group

**Answer:**
```sql
SELECT age, equivalent_cups_group,
       ROUND(COUNT(*) * 1.0 / SUM(COUNT(*)) OVER (PARTITION BY age), 2) AS percentage
FROM coffee_cleaned
GROUP BY age, equivalent_cups_group
ORDER BY CASE age
    WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
  END;
```

**Answer (earlier version, using JOIN instead of a window function):**
```sql
SELECT t.age, s.equivalent_cups_group, s.num_sub, t.num_total,
       ROUND(s.num_sub * 1.0 / t.num_total, 2) AS percentage
FROM (
    SELECT age, COUNT(*) AS num_total
    FROM coffee_cleaned
    GROUP BY age
) t
INNER JOIN (
    SELECT age, equivalent_cups_group, COUNT(*) AS num_sub
    FROM coffee_cleaned
    GROUP BY age, equivalent_cups_group
) s
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
  END;
```

## Analysis 2: Coffee preference by age group
### Q1: Top3 Favorite coffee types by age group

**Answer:**
```sql
WITH favorite_coffee AS (
    SELECT age, favorite, COUNT(*) AS num
    FROM Coffee
    WHERE age != 'NA'
    GROUP BY age, favorite
),
top3 AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY age
        ORDER BY num DESC
    ) AS rank
    FROM favorite_coffee
)
SELECT age,
    MAX(CASE WHEN rank = 1 THEN favorite END) AS top1,
    MAX(CASE WHEN rank = 2 THEN favorite END) AS top2,
    MAX(CASE WHEN rank = 3 THEN favorite END) AS top3
FROM top3
GROUP BY age
ORDER BY CASE age
    WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
  END;
```

| age              | top1                 | top2      | top3                 |
|------------------|----------------------|-----------|----------------------|
| <18 years old    | Latte                | Pourover  | Regular drip coffee  |
| 18-24 years old  | Pourover             | Latte     | Cortado              |
| 25-34 years old  | Pourover             | Latte     | Regular drip coffee  |
| 35-44 years old  | Pourover             | Latte     | Regular drip coffee  |
| 45-54 years old  | Pourover             | Latte     | Regular drip coffee  |
| 55-64 years old  | Regular drip coffee  | Pourover  | Latte                |
| >65 years old    | Regular drip coffee  | Pourover  | Latte                |

**Answer (earlier version):**
```sql
WITH favorite_coffee AS (
    SELECT age, favorite, COUNT(*) AS num
    FROM Coffee
    WHERE age != 'NA'
    GROUP BY age, favorite
)
SELECT age, favorite, num
FROM (
    SELECT *, RANK() OVER (
        PARTITION BY age
        ORDER BY num DESC
    ) AS rank
    FROM favorite_coffee
) t
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
  END, num DESC;
```

**Findings:**
- Younger people prefer `Latte` more than older people.

### Hypothesis: Do younger people prefer coffee which is smoother and less bitter?

**Answer:**
```sql
SELECT age,
    MAX(CASE WHEN sugar_dummy = 'Y' THEN percentage END) AS with_sugar,
    MAX(CASE WHEN sugar_dummy = 'N' THEN percentage END) AS without_sugar
FROM (
    SELECT age, sugar_dummy,
        ROUND(COUNT(*) * 1.0 / SUM(COUNT(*)) OVER (PARTITION BY age), 2) AS percentage
    FROM coffee_cleaned
    GROUP BY age, sugar_dummy
) t
GROUP BY age
ORDER BY CASE age
    WHEN '<18 years old' THEN 1
    WHEN '18-24 years old' THEN 2
    WHEN '25-34 years old' THEN 3
    WHEN '35-44 years old' THEN 4
    WHEN '45-54 years old' THEN 5
    WHEN '55-64 years old' THEN 6
    WHEN '>65 years old' THEN 7
    ELSE 8
  END;
```

| age | with_sugar | without_sugar |
|---|---|---|
| <18 years old | 0.53 | 0.47 |
| 18-24 years old | 0.24 | 0.76 |
| 25-34 years old | 0.14 | 0.86 |
| 35-44 years old | 0.15 | 0.85 |
| 45-54 years old | 0.19 | 0.81 |
| 55-64 years old | 0.23 | 0.77 |
| >65 years old | 0.14 | 0.86 |

**Findings:**
- Sugar/syrup use is highest among under-18s, drops to a low around 25–34, then rises again through middle age.
- Younger and under-18 groups show both a higher preference for Latte and a higher rate of adding sugar/syrup.
- Whether these are causally linked (e.g. a shared preference for less bitter coffee) cannot be determined from this data.

## Analysis 3: Coffee preference by races

**Data validation and cleaning:**
```sql
SELECT ethnicity_race, COUNT(*) FROM Coffee GROUP BY ethnicity_race;

DROP VIEW IF EXISTS coffee_preferences;
CREATE VIEW coffee_preferences AS
    SELECT
        ethnicity_race,
        favorite,
        additions,
        CASE
            WHEN additions LIKE '%Flavor syrup%' THEN 'Y'
            WHEN additions LIKE '%Sugar or sweetener%' THEN 'Y'
            ELSE 'N'
        END AS sugar_dummy,
        style,
        strength,
        roast_level
    FROM Coffee
    WHERE ethnicity_race <> 'NA';

SELECT DISTINCT * FROM coffee_preferences LIMIT 50;
```

**Notes:**
- The number of people who chose `Other (please specify)` as their `ethnicity_race` is very small (less than 3 people per race), so other races won't be analyzed in this report.

### Q1: Favorite coffee types by races

**Answer:**
```sql
WITH favorite_rank AS (
    SELECT
        ethnicity_race,
        favorite,
        ROW_NUMBER() OVER (
            PARTITION BY ethnicity_race
            ORDER BY num DESC
        ) AS rank
    FROM (
        SELECT ethnicity_race, favorite, COUNT(*) AS num
        FROM coffee_preferences
        GROUP BY ethnicity_race, favorite
    )
)
SELECT ethnicity_race,
    MAX(CASE WHEN rank = 1 THEN favorite END) AS first,
    MAX(CASE WHEN rank = 2 THEN favorite END) AS second,
    MAX(CASE WHEN rank = 3 THEN favorite END) AS third
FROM favorite_rank
GROUP BY ethnicity_race;
```

**Answer (earlier version):**
```sql
WITH favorite_count AS (
    SELECT ethnicity_race, favorite, COUNT(*) AS num
    FROM coffee_preferences
    GROUP BY ethnicity_race, favorite
),
favorite_rank AS (
    SELECT
        ethnicity_race,
        favorite,
        ROW_NUMBER() OVER (
            PARTITION BY ethnicity_race
            ORDER BY num DESC
        ) AS rank
    FROM favorite_count
)
SELECT
    r1.ethnicity_race,
    r1.favorite AS first,
    r2.favorite AS second,
    r3.favorite AS third
FROM favorite_rank r1
INNER JOIN favorite_rank r2
    ON r1.ethnicity_race = r2.ethnicity_race
INNER JOIN favorite_rank r3
    ON r2.ethnicity_race = r3.ethnicity_race
WHERE r1.rank = 1
    AND r2.rank = 2
    AND r3.rank = 3;
```

| ethnicity_race | first | second | third |
|---|---|---|---|
| Asian/Pacific Islander | Pourover | Latte | Cappuccino |
| Black/African American | Pourover | Latte | Cappuccino |
| Hispanic/Latino | Pourover | Latte | Espresso |
| Native American/Alaska Native | Latte | Mocha | Regular drip coffee |
| Other (please specify) | Pourover | Latte | Cappuccino |
| White/Caucasian | Pourover | Latte | Regular drip coffee |

**Findings:**
- `Pourover` and `Latte` are the top2 favorite coffee types for all the races except `Native American/Alaska Native`, whose favorite coffee styles are `Latte` and `Mocha`.

### Q2: Most popular strength, style, and roast level independently by race

**Answer:**
```sql
WITH strength_cte AS (
    SELECT ethnicity_race, strength,
        ROW_NUMBER() OVER (PARTITION BY ethnicity_race ORDER BY strength_num DESC) AS strength_rank
    FROM (
        SELECT ethnicity_race, strength, COUNT(*) AS strength_num
        FROM coffee_preferences
        GROUP BY ethnicity_race, strength
    )
),
style_cte AS (
    SELECT ethnicity_race, style,
        ROW_NUMBER() OVER (PARTITION BY ethnicity_race ORDER BY style_num DESC) AS style_rank
    FROM (
        SELECT ethnicity_race, style, COUNT(*) AS style_num
        FROM coffee_preferences
        GROUP BY ethnicity_race, style
    )
),
roast_level_cte AS (
    SELECT ethnicity_race, roast_level,
        ROW_NUMBER() OVER (PARTITION BY ethnicity_race ORDER BY roast_level_num DESC) AS roast_level_rank
    FROM (
        SELECT ethnicity_race, roast_level, COUNT(*) AS roast_level_num
        FROM coffee_preferences
        GROUP BY ethnicity_race, roast_level
    )
)
SELECT st.ethnicity_race, st.strength, s.style, r.roast_level
FROM strength_cte st
INNER JOIN style_cte s ON st.ethnicity_race = s.ethnicity_race
INNER JOIN roast_level_cte r ON st.ethnicity_race = r.ethnicity_race
WHERE st.strength_rank = 1
    AND s.style_rank = 1
    AND r.roast_level_rank = 1;
```

| ethnicity_race | strength | style | roast_level |
|---|---|---|---|
| Asian/Pacific Islander | Somewhat strong | Fruity | Light |
| Black/African American | Medium | Sweet | Medium |
| Hispanic/Latino | Medium | Chocolatey | Medium |
| Native American/Alaska Native | Somewhat strong | Bold | Dark |
| Other (please specify) | Somewhat strong | Fruity | Light |
| White/Caucasian | Somewhat strong | Fruity | Light |

**Findings:**
- Different races have different preference for `style`, `strength`, and `roast_level`.
- This result could be helpful if someone is going to open a coffee store targeting a specific demographic.
- For example: if I want to open a store at an area with many `Asian` tourists, I will prepare coffee with `fruity` and `light` and `somewhat strong taste`, assuming the survey sample is representative.
