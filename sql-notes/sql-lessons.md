# SQL Lessons

Personal reference notes collected while practicing SQL on real datasets
(e-Stat employment data, Coffee Taste Test Survey). Grouped by theme.
Each note is a trigger — the shortest phrase that lets me recall the full rule.

---

## 1. Window functions

- **ROW_NUMBER() / RANK() must include ORDER BY.** Without it the ranking is
  undefined (effectively random). It won't raise an error and may even look
  correct by chance — the most dangerous kind of bug.

- **Pick the right ranking function by how it handles ties:**
  - `ROW_NUMBER()` — no ties, one row per rank (use for "strictly N rows per group").
  - `RANK()` — ties share a rank, then skip (1,1,3). May return more than N rows.
  - `DENSE_RANK()` — ties share a rank, no skip (1,1,2).

- **Difference from the base row:**
  - minus previous row → `LAG(value) OVER (ORDER BY ...)`
  - minus first row → `FIRST_VALUE(value) OVER (ORDER BY ...)`
  - "vs. baseline / vs. starting point" (基準年比) → FIRST_VALUE
  - "vs. previous period / month-over-month" (前月比) → LAG

- **Within-group percentage:** `SUM(COUNT(*)) OVER (PARTITION BY group)`
  is cleaner than a self-JOIN on subqueries. Pattern: detail / window-total.

---

## 2. GROUP BY & aggregation

- **Watch out for missing GROUP BY columns.** After GROUP BY, SELECT may only
  contain (a) grouped columns or (b) columns wrapped in an aggregate. Any other
  "bare" column errors in most databases — but SQLite silently returns a random
  value from the group, which looks like data but is meaningless.

- **Pull values from different rows onto one row with conditional aggregation:**
  `MAX(CASE WHEN condition THEN value END)`. MAX ignores NULL, so it "picks out"
  the single non-NULL value. Used for pivoting, ratios, "put male/female side by
  side", "one row per year".

- **Lay out a group's Top-N as columns:** a self-JOIN silently drops the entire
  group when fewer than N rows exist; conditional aggregation
  (`MAX(CASE WHEN rank = n ...)`) keeps the row and shows NULL for missing ranks.
  The latter is safer.

- **Integer division truncates.** `300/500` returns `0`, not `0.6`. Multiply by
  `1.0` or CAST to REAL before dividing.

---

## 3. Data cleaning

- **Before mapping with CASE, run `SELECT DISTINCT` first** to confirm every
  value is covered. Unlisted values silently become NULL and get dropped.

- **`'NA'` (text) and `NULL` (real null) are different things.**
  `WHERE col <> 'NA'` does not filter real NULLs. Check with `SELECT DISTINCT`
  what forms of "missing" actually exist (text 'NA', empty string, real NULL).

- **Encoding vs. delimiter symptoms.** If columns split correctly but Japanese
  text is garbled → encoding problem (Shift-JIS read as UTF-8); fix by
  re-saving as UTF-8 in Excel. If everything is crammed into one column →
  delimiter problem. Two different failures, two different fixes.

- **e-Stat downloads:** choose 列指向形式 (long format), not クロス集計表形式
  (cross-tab). Long format is what SQL wants — one record per row.

- **Document any assumptions.** E.g. "'More than 4' approximated as 5". Separate
  "the data" from "my estimate" — this is analytical honesty, not a footnote.

---

## 4. Sorting & display

- **Sort text categories in a meaningful (non-alphabetical) order** with
  `ORDER BY CASE column WHEN ... THEN number END`. Needed for age brackets,
  size labels, etc. that have semantic order but are stored as text.

- **When copying a CASE row (first/second/third, Jan/Feb/Mar), read the numbers
  down the column afterward** — "1, 2, 3" — to catch copy-paste duplicates.

---

## 5. Analytical rigor

- **Correlation ≠ causation.** Data showing A and B move together does not mean
  A causes B. Before concluding, ask: could a third factor drive both? Word it
  as "associated with" / "tends to", not "because".

- **Small samples aren't a keep/drop decision** but a question of how specific a
  conclusion they can support: the smaller n is, the coarser the claim must be.
  Labeling n lets readers judge for themselves — more honest than deciding for
  them. The finer the breakdown (multi-dimensional splits), the more a small n
  gets stretched.

- **Most popular value per dimension ≠ most popular combination.** Taking each
  dimension's mode separately and stitching them together can produce a
  combination that no one actually chose. For a real combination, GROUP BY all
  dimensions together and count.

- **Verify aggregates against raw detail.** Pull one group's underlying rows and
  hand-check that the aggregate matches. Builds trust in the result.

---

## 6. Structure & workflow

- **An alias defined in SELECT can't be referenced elsewhere in the same SELECT**
  (e.g. in WHERE or another SELECT expression), so use a CTE. Exception: ORDER BY
  *can* use SELECT aliases, because it runs last.

- **SQL execution order:**
  FROM → WHERE → GROUP BY → aggregate functions → SELECT →
  window functions (ROW_NUMBER(), RANK()) → ORDER BY.
  This one order explains most of the traps above: WHERE can't see window
  functions or SELECT aliases (both run later); ORDER BY can (it runs last).

- **`CREATE VIEW` / `CREATE TABLE AS` to fix a cleaning or derivation layer**,
  keeping the raw table untouched. Separates "raw" from "analysis" — standard
  practice. Prefix with `DROP VIEW/TABLE IF EXISTS` to rebuild freely.

- **JOIN type signals intent.** When sources are equivalent, prefer INNER/LEFT
  over RIGHT to avoid confusing readers.

- **Convert e-Stat `time_code` to a year:** `time_code / 1000000` (numeric) or
  `SUBSTR(time_code, 1, 4)` (string, wrap in CAST for math).

---

## 7. Recurring habit (the meta-lesson)

Before writing any non-trivial query, spend 30 seconds on
`SELECT * FROM table LIMIT 5;` and `SELECT DISTINCT col FROM table;` to confirm
real column names, data grain, encoding, and what values exist. Most bugs above
are prevented by looking before writing. Survey the scene before deducing.
