# SQL Lessons (EN / 日本語 / 中文)

Personal reference notes from practicing SQL on real datasets (e-Stat employment
data, Coffee Taste Test Survey). Each note appears in three languages for
interview prep. Order per note: **EN → JA → ZH**.

実データ（e-Statの就業データ、Coffee Taste Test Survey）でSQLを練習しながら
まとめた個人メモ。面接準備のため3言語で記載。各項目は EN → JA → ZH の順。

用真实数据集（e-Stat就业数据、Coffee口味调查）练习SQL时整理的个人笔记。
为面试准备用三种语言记录。每条顺序为 英 → 日 → 中。

---

## 1. Window functions / ウィンドウ関数 / 窗口函数

**ROW_NUMBER() / RANK() must include ORDER BY**, or the ranking is undefined
(effectively random). It won't error and may even look correct by chance — the
most dangerous kind of bug.
🇯🇵 **ROW_NUMBER() / RANK() には必ず ORDER BY を付ける。** ないと順位は未定義
（実質ランダム）になる。エラーは出ず、たまたま正しく見えることもあるため、
最も危険なバグ。
🇨🇳 **ROW_NUMBER() / RANK() 必须带 ORDER BY**，否则名次是未定义的（实际随机）。
它不会报错，甚至可能碰巧看着正确——最危险的一类 bug。

---

**Pick the ranking function by how it handles ties:** ROW_NUMBER (no ties, one
row per rank), RANK (ties share a rank, then skip: 1,1,3), DENSE_RANK (ties
share, no skip: 1,1,2).
🇯🇵 **同順位の扱いで関数を選ぶ:** ROW_NUMBER（同順位なし・1グループN行）、
RANK（同順位あり・その後スキップ 1,1,3）、DENSE_RANK（同順位あり・スキップ
なし 1,1,2）。
🇨🇳 **按如何处理并列来选函数:** ROW_NUMBER（无并列、每组恰N行）、RANK（并列
同名次、之后跳号 1,1,3）、DENSE_RANK（并列同名次、不跳号 1,1,2）。

---

**Difference from a base row:** minus previous row → LAG; minus first row →
FIRST_VALUE. "vs. baseline"(基準年比) → FIRST_VALUE; "month-over-month"(前月比)
→ LAG.
🇯🇵 **基準行との差分:** 前の行との差 → LAG、最初の行との差 → FIRST_VALUE。
「基準年比」→ FIRST_VALUE、「前月比」→ LAG。
🇨🇳 **与基准行的差:** 减上一行 → LAG、减第一行 → FIRST_VALUE。
「基准年比」→ FIRST_VALUE、「前月比/环比」→ LAG。

---

**Within-group percentage:** SUM(COUNT(*)) OVER (PARTITION BY group) is cleaner
than a self-JOIN on subqueries. Pattern = detail / window-total.
🇯🇵 **グループ内の割合:** SUM(COUNT(*)) OVER (PARTITION BY グループ) は
サブクエリの自己結合より簡潔。パターン = 明細 / ウィンドウ合計。
🇨🇳 **组内占比:** SUM(COUNT(*)) OVER (PARTITION BY 组) 比子查询自连接更简洁。
模式 = 明细 / 窗口总计。

---

## 2. GROUP BY & aggregation / GROUP BY と集約 / GROUP BY 与聚合

**Watch out for missing GROUP BY columns.** After GROUP BY, SELECT may only
contain grouped columns or aggregated columns. A bare column errors in most
databases, but SQLite silently returns a random value from the group.
🇯🇵 **GROUP BY の列漏れに注意。** GROUP BY 後の SELECT には、グループ化した列か
集約した列しか置けない。素の列は多くのDBでエラーになるが、SQLiteはグループ内
から適当な値を黙って返す。
🇨🇳 **注意 GROUP BY 漏列。** GROUP BY 后的 SELECT 只能放分组列或聚合列。裸列在
多数数据库会报错，但 SQLite 会悄悄返回组内的随机值。

---

**Pull values from different rows onto one row with conditional aggregation:**
MAX(CASE WHEN condition THEN value END). MAX ignores NULL, so it picks out the
single non-NULL value. Used for pivoting, ratios, side-by-side comparisons.
🇯🇵 **異なる行の値を1行にまとめる条件付き集約:**
MAX(CASE WHEN 条件 THEN 値 END)。MAX は NULL を無視するので、唯一の非NULL値を
取り出せる。ピボット、比率、横並び比較に使う。
🇨🇳 **用条件聚合把不同行的值收拢到一行:** MAX(CASE WHEN 条件 THEN 值 END)。
MAX 忽略 NULL，因此能取出唯一的非 NULL 值。用于行转列、比率、横向对比。

---

**Lay out a group's Top-N as columns:** a self-JOIN silently drops the whole
group when fewer than N rows exist; conditional aggregation
(MAX(CASE WHEN rank = n ...)) keeps the row and shows NULL. The latter is safer.
🇯🇵 **グループのTop-Nを列に展開:** 自己結合はN行に満たないグループを黙って
丸ごと落とす。条件付き集約（MAX(CASE WHEN rank = n ...)）は行を残しNULLを表示。
後者が安全。
🇨🇳 **把组内 Top-N 排成列:** 自连接在凑不齐 N 行时会静默丢掉整组；条件聚合
（MAX(CASE WHEN rank = n ...)）会保留行并显示 NULL。后者更安全。

---

**Integer division truncates.** 300/500 returns 0, not 0.6. Multiply by 1.0 or
CAST to REAL before dividing.
🇯🇵 **整数の除算は切り捨て。** 300/500 は 0.6 ではなく 0 になる。割る前に 1.0 を
掛けるか REAL に CAST する。
🇨🇳 **整数相除会截断。** 300/500 得到 0 而非 0.6。相除前乘 1.0 或 CAST 成 REAL。

---

## 3. Data cleaning / データクレンジング / 数据清洗

**Before mapping with CASE, run SELECT DISTINCT first** to confirm every value is
covered. Unlisted values silently become NULL and get dropped.
🇯🇵 **CASE でマッピングする前に、まず SELECT DISTINCT** で全ての値が網羅されて
いるか確認する。列挙し忘れた値は黙って NULL になり脱落する。
🇨🇳 **用 CASE 映射前，先跑 SELECT DISTINCT** 确认所有值都被覆盖。没列进去的值
会悄悄变成 NULL 被丢掉。

---

**'NA' (text) and NULL (real null) are different.** WHERE col <> 'NA' does not
filter real NULLs. Check with SELECT DISTINCT what forms of "missing" exist.
🇯🇵 **'NA'（文字列）と NULL（真の空値）は別物。** WHERE col <> 'NA' では本当の
NULL は除外されない。SELECT DISTINCT で欠損の形（'NA'、空文字、NULL）を確認する。
🇨🇳 **'NA'（文本）和 NULL（真空值）是两回事。** WHERE col <> 'NA' 挡不住真正的
NULL。用 SELECT DISTINCT 确认缺失有哪些形式（'NA'、空字符串、NULL）。

---

**Encoding vs. delimiter symptoms.** Columns split correctly but Japanese text
garbled → encoding problem (fix: re-save as UTF-8 in Excel). Everything crammed
into one column → delimiter problem. Two failures, two fixes.
🇯🇵 **文字化けと区切り文字の症状の違い。** 列は正しく分かれているのに日本語が
文字化け → エンコーディング問題（Excelで UTF-8 保存し直す）。全部が1列に
詰まる → 区切り文字問題。原因が違えば対処も違う。
🇨🇳 **编码问题 vs 分隔符问题的症状。** 分列正常但日文乱码 → 编码问题
（用 Excel 另存为 UTF-8）。所有内容挤在一列 → 分隔符问题。两种故障，两种解法。

---

**e-Stat downloads:** choose 列指向形式 (long format), not クロス集計表形式
(cross-tab). Long format is what SQL wants — one record per row.
🇯🇵 **e-Stat のダウンロード:** クロス集計表形式ではなく、列指向形式（ロング
フォーマット）を選ぶ。1行1レコードの列指向形式こそSQL向き。
🇨🇳 **e-Stat 下载:** 选「列指向形式」（长格式），不要「クロス集計表形式」
（交叉表）。长格式才是 SQL 想要的——每行一条记录。

---

**Document any assumptions** (e.g. "'More than 4' approximated as 5"). Separate
"the data" from "my estimate" — analytical honesty.
🇯🇵 **仮定は必ず明記する**（例:「'More than 4' を 5 と近似」）。「データ」と
「自分の推定」を分ける — 分析の誠実さ。
🇨🇳 **任何假设都要写明**（如「'More than 4' 近似为 5」）。把「数据」和
「我的估计」分开——分析的诚实性。

---

## 4. Sorting & display / 並び替えと表示 / 排序与显示

**Sort text categories in a meaningful (non-alphabetical) order** with
ORDER BY CASE column WHEN ... THEN number END. Needed for age brackets, size
labels — semantic order stored as text.
🇯🇵 **テキストのカテゴリを意味のある順で並べる**（アルファベット順ではなく）:
ORDER BY CASE 列 WHEN ... THEN 数値 END。年齢層やサイズ表記など、文字列だが
意味上の順序を持つものに必要。
🇨🇳 **文本类别按语义顺序排**（而非字母顺序）: ORDER BY CASE 列 WHEN ... THEN
数字 END。年龄段、尺寸标签等「有语义顺序却存成文本」的情况需要。

---

**When copying a CASE row (first/second/third, Jan/Feb/Mar), read the numbers
down the column** — "1, 2, 3" — to catch copy-paste duplicates.
🇯🇵 **CASE の行をコピーしたら（first/second/third、1月/2月/3月）、数字を上から
読み下す** — 「1, 2, 3」— コピペの重複を見つける。
🇨🇳 **复制 CASE 行后（first/second/third、1月/2月/3月），把数字从上往下念一遍**
——「1、2、3」——抓出复制粘贴的重复。

---

## 5. Analytical rigor / 分析の厳密さ / 分析严谨性

**Correlation ≠ causation.** A and B moving together doesn't mean A causes B.
Ask: could a third factor drive both? Word it as "associated with" / "tends to",
not "because".
🇯🇵 **相関 ≠ 因果。** A と B が一緒に動いても A が B の原因とは限らない。
第3の要因が両方を動かしていないか?と問う。「〜のため（because）」ではなく
「〜と関連する / 〜の傾向がある」と表現する。
🇨🇳 **相关 ≠ 因果。** A 和 B 一起变化不代表 A 导致 B。要问:有没有第三个因素同时
影响两者?措辞用「与…相关 / 倾向于」，而非「因为」。

---

**Small samples aren't a keep/drop decision** but a question of how specific a
conclusion they support: the smaller n, the coarser the claim. Labeling n lets
readers judge — more honest than deciding for them. Finer breakdowns stretch a
small n further.
🇯🇵 **小さいサンプルは「残す/捨てる」の二択ではなく**、どこまで細かい結論を
支えられるかの問題。n が小さいほど主張は粗くすべき。n を明記して読者に判断を
委ねる方が、代わりに決めるより誠実。多次元の細分化ほど小さい n に負担がかかる。
🇨🇳 **小样本不是「删不删」的二选一**，而是「能支撑多精细的结论」的问题:n 越小，
结论越要粗。标注 n 让读者自己判断，比替他们决定更诚实。越是多维度细分，越会
放大小样本的问题。

---

**Most popular value per dimension ≠ most popular combination.** Taking each
dimension's mode separately and stitching them can produce a combination no one
chose. For a real combination, GROUP BY all dimensions together and count.
🇯🇵 **各次元の最頻値 ≠ 最頻の組み合わせ。** 次元ごとに最頻値を取って繋げると、
誰も選んでいない組み合わせができうる。本当の組み合わせは、全次元を GROUP BY
してまとめて数える。
🇨🇳 **各维度的众数 ≠ 最常见的组合。** 分别取每个维度的众数再拼起来，可能得到
没人选过的组合。要真组合，就 GROUP BY 所有维度一起数。

---

**Verify aggregates against raw detail.** Pull one group's underlying rows and
hand-check the aggregate matches. Builds trust in the result.
🇯🇵 **集約結果を生データで検証する。** あるグループの元の行を取り出し、集約値と
一致するか手で確認する。結果への信頼につながる。
🇨🇳 **用原始明细验证聚合结果。** 调出某组的底层行，手动核对聚合值是否吻合。
建立对结果的信任。

---

## 6. Structure & workflow / 構造とワークフロー / 结构与工作流

**An alias defined in SELECT can't be referenced elsewhere in the same SELECT**
(e.g. in WHERE), so use a CTE. Exception: ORDER BY can use SELECT aliases,
because it runs last.
🇯🇵 **SELECT 内で付けた別名は同じ SELECT の他の場所（WHERE など）で参照でき
ない**ため、CTE を使う。例外: ORDER BY は最後に実行されるので別名を使える。
🇨🇳 **在 SELECT 里定义的别名，不能在同一个 SELECT 的其他地方（如 WHERE）引用**，
所以用 CTE。例外: ORDER BY 能用 SELECT 别名，因为它最后执行。

---

**SQL execution order:** FROM → WHERE → GROUP BY → aggregate functions → SELECT
→ window functions → ORDER BY. This explains most traps: WHERE can't see window
functions or SELECT aliases (both run later); ORDER BY can (it runs last).
🇯🇵 **SQLの実行順序:** FROM → WHERE → GROUP BY → 集約関数 → SELECT →
ウィンドウ関数 → ORDER BY。多くの罠はこれで説明できる: WHERE は後で実行される
ウィンドウ関数や SELECT の別名を参照できない。ORDER BY は最後なので参照できる。
🇨🇳 **SQL 执行顺序:** FROM → WHERE → GROUP BY → 聚合函数 → SELECT → 窗口函数
→ ORDER BY。多数坑都源于此: WHERE 看不到窗口函数和 SELECT 别名（都在它之后
执行）；ORDER BY 能（它最后执行）。

---

**CREATE VIEW / CREATE TABLE AS to fix a cleaning or derivation layer**, keeping
the raw table untouched. Separates raw from analysis. Prefix with
DROP ... IF EXISTS to rebuild freely.
🇯🇵 **CREATE VIEW / CREATE TABLE AS でクレンジング・派生レイヤーを固定し**、
生テーブルはそのまま残す。生データと分析を分離。DROP ... IF EXISTS を先に
付ければ何度でも作り直せる。
🇨🇳 **用 CREATE VIEW / CREATE TABLE AS 固化清洗或派生层**，原始表保持不动。
把原始数据和分析分离。前面加 DROP ... IF EXISTS 可反复重建。

---

**JOIN type signals intent.** When sources are equivalent, prefer INNER/LEFT
over RIGHT to avoid confusing readers.
🇯🇵 **JOIN の種類は意図を伝える。** ソースが等価なら、読み手を混乱させない
よう RIGHT より INNER/LEFT を選ぶ。
🇨🇳 **JOIN 类型传递意图。** 数据源等价时，用 INNER/LEFT 而非 RIGHT，避免让读者
困惑。

---

**Convert e-Stat time_code to a year:** time_code / 1000000 (numeric) or
SUBSTR(time_code, 1, 4) (string; CAST for math).
🇯🇵 **e-Stat の time_code を年に変換:** time_code / 1000000（数値）または
SUBSTR(time_code, 1, 4)（文字列。計算するなら CAST）。
🇨🇳 **把 e-Stat 的 time_code 转成年份:** time_code / 1000000（数值）或
SUBSTR(time_code, 1, 4)（字符串；要计算则 CAST）。

---

## 7. The meta-lesson / メタ教訓 / 元教训

**Before any non-trivial query, spend 30 seconds** on SELECT * ... LIMIT 5 and
SELECT DISTINCT col to confirm real column names, data grain, encoding, and
existing values. Most bugs above are prevented by looking before writing.
🇯🇵 **少し複雑なクエリを書く前に、30秒だけ** SELECT * ... LIMIT 5 と
SELECT DISTINCT col を実行し、実際の列名・データの粒度・エンコーディング・存在
する値を確認する。上記のバグの多くは「書く前に見る」だけで防げる。
🇨🇳 **写任何稍复杂的查询前，花 30 秒** 跑 SELECT * ... LIMIT 5 和
SELECT DISTINCT col，确认真实列名、数据粒度、编码和现有取值。上面多数 bug，
靠「动手前先看」就能提前挡掉。
