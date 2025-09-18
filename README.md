## Repsol-Research-Analysis

### Project Overview
Comparative analysis of Sporting CP, SC Braga, and Vitória SC during Liga Portugal 2021–22 to evaluate sponsorship ROI, fan engagement, and attendance, with insights on which club offers Repsol the strongest partnership potential.

---
<img width="1210" height="674" alt="Betclic Liga" src="https://github.com/user-attachments/assets/53d3340e-cf99-45b4-8c46-a093c0d7dbff" />


### Data Source
The dataset consists of official 2022 season metrics for SC Braga, Sporting CP, and Vitória SC. It includes key performance indicators such as attendance, digital engagement, and revenue-related figures. These metrics were collected to evaluate the return on investment (ROI) potential for Repsol’s partnership strategy.

### Tools

- Excel - Data Cleanning [Download here](https://1drv.ms/x/c/727c76b1462e1994/EVC1gI6HpAhMq8Zf80Db4isB7gU_n07l1NsJA6HzvxPw3g?e=GMhNo0) 
- [Download here](https://1drv.ms/x/c/727c76b1462e1994/EZziOHCF3stOiLZV2GkvXy4BfdHIr44zVUouUs5kjzr6Hg?e=vMe5OB) 
- [Download here](https://1drv.ms/x/c/727c76b1462e1994/ER6iBfNPIQdEiHuW6ATiUfQB0cH_Iq0aQyJpJsKfRCngSw?e=KiXPIB)
- SQL Server - Data Analysis [Download here](https://1drv.ms/u/c/29f0e449ed577bcc/EQ4XJjSTE3ZKpO4c2-pJhnQB3tWdNR-4Rgp7l4J9bUA-Og?e=3HnJZj)
- Power BI - Creating Reports [Download here]()

### Data Cleaning & Preparation

The raw datasets from SC Braga, Sporting CP, and Vitória SC contained season metrics in varying formats. To ensure consistency and accuracy across all clubs, the following steps were carried out:

1. Standardized column headers – Renamed inconsistent fields (e.g., “Daily Unique Visits” vs. “Unique Visits”) for uniformity across files. 
2. Handled missing values – Imputed or removed null entries in attendance, revenue, and engagement fields.
3. Date formatting – Converted all match and reporting dates to ISO format (YYYY-MM-DD) for compatibility in SQL Server and Power BI.
4. Data type correction – Ensured numeric fields (attendance, page views, income, ROI) were properly cast as integers/decimals.
5. Derived metrics – Created new calculated columns such as:

CPM ($) = (Daily Website Income / Daily Page Views) × 1000
EPMV ($) = (Daily Website Income / Daily Unique Visits) × 1000
Season Revenue ($) = SUM(Daily Website Income)

6. Merged datasets – Combined the three club files into a unified structure for cross-club comparison.
7. Validation checks – Verified totals (attendance per season, average occupancy rates, and revenue sums) against source reports to ensure accuracy.                                                
### Data Analysis

My analysis code/features i worked with:

1. Row counts & date coverage by club ? 
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club, COUNT(*) AS rows_, MIN([date]) AS first_date, MAX([date]) AS last_date FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP', COUNT(*), MIN([date]), MAX([date]) FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC', COUNT(*), MIN([date]), MAX([date]) FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY club;

2. Core totals (unique visits, pageviews, income, season revenue) ?
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club,
         SUM(daily_unique_visits) AS total_unique_visits,
         SUM(daily_page_views)    AS total_page_views,
         SUM(daily_website_income) AS total_income,
         SUM(season_revenue)       AS total_season_revenue
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         SUM(daily_unique_visits), SUM(daily_page_views),
         SUM(daily_website_income), SUM(season_revenue)
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         SUM(daily_unique_visits), SUM(daily_page_views),
         SUM(daily_website_income), SUM(season_revenue)
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY total_income DESC;

3. Weighted CPM & EPMV (income-weighted is correct)?
```sql
 */
WITH t AS (
  SELECT 'SC Braga' AS club,
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0)     AS weighted_cpm,
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0)  AS weighted_epmv
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0),
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0),
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY weighted_epmv DESC;

4. Average day performance per club? 
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club,
         AVG(CAST(daily_website_income AS decimal(18,6))) AS avg_daily_income,
         AVG(CAST(daily_page_views     AS decimal(18,6))) AS avg_daily_page_views,
         AVG(CAST(daily_unique_visits  AS decimal(18,6))) AS avg_daily_unique_visits
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         AVG(CAST(daily_website_income AS decimal(18,6))),
         AVG(CAST(daily_page_views     AS decimal(18,6))),
         AVG(CAST(daily_unique_visits  AS decimal(18,6)))
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         AVG(CAST(daily_website_income AS decimal(18,6))),
         AVG(CAST(daily_page_views     AS decimal(18,6))),
         AVG(CAST(daily_unique_visits  AS decimal(18,6)))
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY avg_daily_income DESC;

5. Median daily income (robust to outliers) ?
```sql
 */
WITH t AS (
  SELECT DISTINCT 'SC Braga' AS club,
         PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY daily_website_income)
           OVER () AS median_daily_income
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT DISTINCT 'Sporting CP',
         PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY daily_website_income)
           OVER () 
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT DISTINCT N'Vitória SC',
         PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY daily_website_income)
           OVER () 
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY median_daily_income DESC;

6. Volatility: std dev of daily income ?
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club, STDEV(CAST(daily_website_income AS float)) AS income_stddev FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP', STDEV(CAST(daily_website_income AS float)) FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC', STDEV(CAST(daily_website_income AS float)) FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY income_stddev ASC;  -- lower = steadier

7. Days beating own average (consistency)?
```sql
 */
WITH b AS (
  SELECT 'SC Braga' AS club, daily_website_income,
         AVG(daily_website_income) OVER () AS club_avg
  FROM dbo.scbraga_pt
), s AS (
  SELECT 'Sporting CP' AS club, daily_website_income,
         AVG(daily_website_income) OVER () AS club_avg
  FROM dbo.sportingcp_pt
), v AS (
  SELECT N'Vitória SC' AS club, daily_website_income,
         AVG(daily_website_income) OVER () AS club_avg
  FROM dbo.vitoria_sc_pt
), x AS (
  SELECT * FROM b UNION ALL SELECT * FROM s UNION ALL SELECT * FROM v
)
SELECT club, SUM(CASE WHEN daily_website_income > club_avg THEN 1 ELSE 0 END) AS days_above_avg
FROM x
GROUP BY club
ORDER BY days_above_avg DESC;

8. Top 10 revenue days per club (dates normalized to 2021 or 2022) 
```sql
*/
;WITH t AS (
    SELECT 'SC Braga' AS club, [date], daily_website_income
    FROM (SELECT TOP 10 [date], daily_website_income
          FROM dbo.scbraga_pt ORDER BY daily_website_income DESC) q
    UNION ALL
    SELECT 'Sporting CP', [date], daily_website_income
    FROM (SELECT TOP 10 [date], daily_website_income
          FROM dbo.sportingcp_pt ORDER BY daily_website_income DESC) q
    UNION ALL
    SELECT 'Vitoria SC', [date], daily_website_income
    FROM (SELECT TOP 10 [date], daily_website_income
          FROM dbo.vitoria_sc_pt ORDER BY daily_website_income DESC) q
),
norm_prep AS (
    SELECT
        club,
        [date]              AS original_date,
        daily_website_income,
        CASE WHEN YEAR([date]) <= 2021 THEN 2021 ELSE 2022 END AS norm_year,
        MONTH([date])       AS m,
        DAY([date])         AS d
    FROM t
),
norm AS (
    /* clamp the day if the normalized month in 2021/2022 has fewer days (e.g., 29-Feb) */
    SELECT
        club,
        original_date,
        daily_website_income,
        DATEFROMPARTS(
            norm_year,
            m,
            CASE
                WHEN DAY(EOMONTH(DATEFROMPARTS(norm_year, m, 1))) < d
                     THEN DAY(EOMONTH(DATEFROMPARTS(norm_year, m, 1)))
                ELSE d
            END
        ) AS norm_date
    FROM norm_prep
)
SELECT
    club,
    norm_date AS [date],      -- normalized to 2021 or 2022
    original_date,            -- original date for reference
    daily_website_income
FROM norm
ORDER BY club, daily_website_income DESC;


9. Monthly revenue (1..30th or EOMONTH), by club,
         bucketed as: 2021 = Aug–Dec 2021, 2022 = Jan–Jul 2022 */
```sql

;WITH all_days AS (
    SELECT 'SC Braga'    AS club, [date], daily_website_income FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], daily_website_income FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC'  AS club, [date], daily_website_income FROM dbo.vitoria_sc_pt
),
month_frames AS (
    /* One row per club×month with anchor = 30th (or month end for Feb).
       Keep only Aug–Dec 2021 and Jan–Jul 2022, and tag each month with bucket_year. */
    SELECT DISTINCT
        club,
        CASE
            WHEN YEAR([date]) = 2021 AND MONTH([date]) BETWEEN 8 AND 12 THEN 2021
            WHEN YEAR([date]) = 2022 AND MONTH([date]) BETWEEN 1 AND 7  THEN 2022
            ELSE NULL
        END AS bucket_year,
        DATEFROMPARTS(YEAR([date]), MONTH([date]), 1) AS month_start,
        CASE
            WHEN DAY(EOMONTH([date])) >= 30
                 THEN DATEFROMPARTS(YEAR([date]), MONTH([date]), 30)   -- 30th for 30/31-day months
            ELSE EOMONTH([date])                                      -- last day for Feb
        END AS anchor_date
    FROM all_days
),
monthly AS (
    /* Sum income from day 1 through anchor day for each club×month within the buckets */
    SELECT
        m.club,
        m.bucket_year,
        m.month_start,
        m.anchor_date,
        SUM(d.daily_website_income)                                         AS month_income_1_to_anchor,
        COUNT(*)                                                            AS days_count_present,
        AVG(CAST(d.daily_website_income AS decimal(18,6)))                  AS avg_daily_income_present,
        SUM(d.daily_website_income) * 1.0 /
          NULLIF(CASE WHEN DAY(EOMONTH(m.month_start)) >= 30
                      THEN 30 ELSE DAY(EOMONTH(m.month_start)) END, 0)      AS avg_daily_income_fixed
    FROM month_frames m
    JOIN all_days d
      ON d.club  = m.club
     AND d.[date] >= m.month_start
     AND d.[date] <= m.anchor_date
    WHERE m.bucket_year IN (2021, 2022)
    GROUP BY m.club, m.bucket_year, m.month_start, m.anchor_date
),
bucket_days AS (
    /* Total number of included days per club×bucket (for your “one count per year” requirement) */
    SELECT club, bucket_year, SUM(days_count_present) AS bucket_days_count
    FROM monthly
    GROUP BY club, bucket_year
)
SELECT
    m.club,
    m.bucket_year AS [year],
    DATENAME(MONTH, m.month_start) AS month_name,
    m.month_start,
    m.anchor_date,
    m.month_income_1_to_anchor      AS month_income,
    m.days_count_present,
    b.bucket_days_count,            -- total included days for the entire bucket (per club)
    m.avg_daily_income_present,     -- average over present rows
    m.avg_daily_income_fixed        -- divides by 30 (or Feb’s 28/29)
FROM monthly m
JOIN bucket_days b
  ON b.club = m.club AND b.bucket_year = m.bucket_year
ORDER BY m.club, [year], m.month_start;


11. Best revenue month per club (season Aug 2021 → Jul 2022)? 
```sql
*/
;WITH d AS (
    SELECT 'SC Braga'    AS club, [date], daily_website_income FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], daily_website_income FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC'  AS club, [date], daily_website_income FROM dbo.vitoria_sc_pt
),
norm AS (
    /* Map months to the 2021–2022 season window:
       Aug–Dec → 2021, Jan–Jul → 2022 (keep month, set day=1 for grouping) */
    SELECT
        club,
        DATEFROMPARTS(
            CASE WHEN MONTH([date]) >= 8 THEN 2021 ELSE 2022 END,
            MONTH([date]),
            1
        ) AS month_start,
        daily_website_income
    FROM d
    WHERE MONTH([date]) IN (8,9,10,11,12,1,2,3,4,5,6,7)  -- Aug..Jul
),
m AS (
    SELECT
        club,
        month_start,
        SUM(daily_website_income) AS month_income
    FROM norm
    GROUP BY club, month_start
),
r AS (
    SELECT
        m.*,
        ROW_NUMBER() OVER (PARTITION BY club ORDER BY month_income DESC, month_start) AS rn
    FROM m
)
SELECT
    club,
    DATENAME(MONTH, month_start) AS month_name,
    month_start,
    month_income
FROM r
WHERE rn = 1
ORDER BY month_income DESC;

12. Season totals (Aug 2021 → Jun 2022) with PV/UV and overridden season_revenue?
 */
```sql
;WITH d AS (
    SELECT 'SC Braga'    AS club, [date], daily_website_income, season_revenue,
           daily_page_views, daily_unique_visits
    FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP', [date], daily_website_income, season_revenue,
           daily_page_views, daily_unique_visits
    FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC',  [date], daily_website_income, season_revenue,
           daily_page_views, daily_unique_visits
    FROM dbo.vitoria_sc_pt
),
norm AS (
    -- Keep only Aug..Jun; map months Aug–Dec→2021 and Jan–Jun→2022 (for season window control)
    SELECT
        club,
        DATEFROMPARTS(
            CASE WHEN MONTH([date]) >= 8 THEN 2021 ELSE 2022 END,
            MONTH([date]),
            DAY([date])
        ) AS season_date,
        daily_website_income,
        season_revenue,
        daily_page_views,
        daily_unique_visits
    FROM d
    WHERE MONTH([date]) IN (8,9,10,11,12,1,2,3,4,5,6)
),
agg AS (
    SELECT
        club,
        '2021-2022' AS season_label,
        SUM(CAST(daily_website_income AS decimal(18,2))) AS season_income_total,
        SUM(CAST(daily_page_views     AS bigint))        AS total_page_views,
        SUM(CAST(daily_unique_visits  AS bigint))        AS total_unique_visits
        -- If you want to see the raw table sum for season_revenue, uncomment next line:
        -- ,SUM(CAST(season_revenue AS decimal(18,2)))      AS season_revenue_raw
    FROM norm
    GROUP BY club
),
provided AS (
    -- Override season_revenue with your supplied totals
    SELECT * FROM (VALUES
        ('SC Braga',    CAST(175235.00 AS decimal(18,2))),
        ('Sporting CP', CAST(192250.00 AS decimal(18,2))),
        ('Vitoria SC',  CAST(155220.00 AS decimal(18,2)))
    ) v(club, season_revenue_total)
)
SELECT
    a.club,
    a.season_label,
    CAST(a.season_income_total AS money) AS season_income_total,
    CAST(p.season_revenue_total AS money) AS season_revenue_total,  -- overridden values
    a.total_page_views,
    a.total_unique_visits
    -- If you kept season_revenue_raw above, you can show it here for reference:
    -- ,CAST(a.season_revenue_raw AS money) AS season_revenue_raw
FROM agg a
JOIN provided p
  ON p.club = a.club
ORDER BY a.club;


13 - ROI by club (totals overridden), cost = 125,000
```sql
   Uses your provided totals:
   - SC Braga     = 175,235.00
   - Sporting CP  = 193,150.00
   - Vitória SC   = 160,420.00
   Outputs: total_revenue, sponsor_cost, Profit/Shortfall, gap_to_break_even, ROI % (positive) */

DECLARE @annual_sponsor_cost money = 125000;

WITH totals AS (
    SELECT * FROM (VALUES
        ('SC Braga',      CAST(175235.00 AS decimal(18,2))),
        ('Sporting CP',   CAST(193150.00 AS decimal(18,2))),
        (N'Vitória SC',   CAST(160420.00 AS decimal(18,2)))
    ) v(club, total_revenue)
)
SELECT
    club,
    CAST(total_revenue AS money)                    AS total_revenue,
    @annual_sponsor_cost                            AS sponsor_cost,
    CASE WHEN total_revenue >= @annual_sponsor_cost THEN 'Profit' ELSE 'Shortfall' END AS outcome,
    CAST(ABS(total_revenue - @annual_sponsor_cost) AS money) AS gap_to_break_even,
    CAST(ROUND(ABS((total_revenue - @annual_sponsor_cost) * 100.0
                   / NULLIF(@annual_sponsor_cost,0)), 2) AS decimal(8,2)) AS roi_percent
FROM totals
ORDER BY roi_percent DESC;


14. Rank clubs by ROI (override totals), season_year = 2022, cost = 125,000
```sq
   Totals provided by you:
   - SC Braga     = 163,535.00
   - Sporting CP  = 185,125.00
   - Vitória SC   = 155,120.00
*/

WITH totals AS (
    SELECT * FROM (VALUES
        ('SC Braga',    2022, CAST(163535.00 AS decimal(18,2))),
        ('Sporting CP', 2022, CAST(185125.00 AS decimal(18,2))),
        (N'Vitória SC', 2022, CAST(155120.00 AS decimal(18,2)))
    ) v(club, season_year, total_revenue)
)
SELECT
    club,
    season_year,
    CAST(total_revenue AS money)                        AS total_revenue,
    CAST(125000 AS money)                               AS sponsor_cost,
    CASE WHEN total_revenue >= 125000 THEN 'Profit' ELSE 'Shortfall' END AS outcome,
    CAST(ABS(total_revenue - 125000) AS money)          AS gap_to_break_even,
    CAST(ROUND(ABS((total_revenue - 125000) * 100.0 / 125000.0), 2) AS decimal(8,2)) AS roi_percent
FROM totals
ORDER BY roi_percent DESC;


15. Cumulative season revenue curve (normalize dates to 2021–2022) 
```sql
*/
WITH d AS (
    SELECT 'SC Braga'    AS club, [date], season_revenue FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], season_revenue FROM dbo.sportingcp_pt
    UNION ALL
    SELECT N'Vitória SC' AS club, [date], season_revenue FROM dbo.vitoria_sc_pt
),
normalized AS (
    /* Force Aug–Dec to 2021, Jan–Jun to 2022 */
    SELECT
        club,
        2022 AS season_year,  -- label the season as 2022 (i.e., 2021–22)
        DATEFROMPARTS(
            CASE WHEN MONTH([date]) >= 8 THEN 2021 ELSE 2022 END, -- forced year
            MONTH([date]),
            DAY([date])
        ) AS season_date,
        season_revenue
    FROM d
)
SELECT
    club,
    season_year,
    season_date AS [date],
    SUM(season_revenue) OVER (
        PARTITION BY club, season_year
        ORDER BY season_date
        ROWS UNBOUNDED PRECEDING
    ) AS cum_revenue
FROM normalized
-- keep just the season months Aug..Jun (typical 8–11 + 1–6); remove this WHERE if you want all rows
WHERE MONTH(season_date) IN (8,9,10,11,12,1,2,3,4,5,6)
ORDER BY club, season_date;

16. Break-even date vs annual cost (normalize to 2021–2022, cost = 125,000)
```sql
 */
;WITH d AS (
    SELECT 'SC Braga'    AS club, [date], season_revenue FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], season_revenue FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC'  AS club, [date], season_revenue FROM dbo.vitoria_sc_pt
),
normalized AS (
    -- Force Aug–Dec to 2021; Jan–Jun to 2022; keep only Aug..Jun
    SELECT
        club,
        2022 AS season_year,  -- label the season as 2021–22
        DATEFROMPARTS(
            CASE WHEN MONTH([date]) >= 8 THEN 2021 ELSE 2022 END,
            MONTH([date]),
            DAY([date])
        ) AS season_date,
        season_revenue
    FROM d
    WHERE MONTH([date]) IN (8,9,10,11,12,1,2,3,4,5,6)
),
ord AS (
    SELECT
        club,
        season_year,
        season_date,
        season_revenue,
        SUM(season_revenue) OVER (
            PARTITION BY club, season_year
            ORDER BY season_date
            ROWS UNBOUNDED PRECEDING
        ) AS cum_rev
    FROM normalized
),
first_hit AS (
    SELECT club, season_year, MIN(season_date) AS break_even_date
    FROM ord
    WHERE cum_rev >= 125000
    GROUP BY club, season_year
)
SELECT
    o.club,
    o.season_year,
    fh.break_even_date,
    MAX(o.cum_rev)                                AS final_cum_revenue,
    CAST(125000 AS money)                         AS sponsor_cost,
    CASE WHEN fh.break_even_date IS NULL THEN 'Not reached' ELSE 'Reached' END AS status,
    CAST(ROUND(100.0 * MAX(o.cum_rev) / 125000.0, 2) AS decimal(8,2)) AS pct_of_cost_achieved
FROM ord o
LEFT JOIN first_hit fh
       ON fh.club = o.club AND fh.season_year = o.season_year
GROUP BY o.club, o.season_year, fh.break_even_date
ORDER BY o.club;

17. Pages per visit (depth) 
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club,
         1.0 * SUM(daily_page_views) / NULLIF(SUM(daily_unique_visits),0) AS pages_per_visit
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         1.0 * SUM(daily_page_views) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         1.0 * SUM(daily_page_views) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t ORDER BY pages_per_visit DESC;

18. Efficiency ratio: EPMV-to-CPM 
```sql
*/
WITH w AS (
  SELECT 'SC Braga' AS club,
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0)    AS weighted_cpm,
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0) AS weighted_epmv
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0),
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_page_views),0),
         1000.0 * SUM(daily_website_income) / NULLIF(SUM(daily_unique_visits),0)
  FROM dbo.vitoria_sc_pt
)
SELECT club, weighted_cpm, weighted_epmv,
       weighted_epmv / NULLIF(weighted_cpm,0) AS epmv_to_cpm_ratio
FROM w
ORDER BY epmv_to_cpm_ratio DESC;

19. Club share of total income 
```sql
*/
WITH s AS (
  SELECT 'SC Braga' AS club, SUM(daily_website_income) AS club_income FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP', SUM(daily_website_income) FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC', SUM(daily_website_income) FROM dbo.vitoria_sc_pt
), total AS (SELECT SUM(club_income) AS all_income FROM s)
SELECT s.club, s.club_income,
       100.0 * s.club_income / NULLIF(total.all_income,0) AS income_share_pct
FROM s CROSS JOIN total
ORDER BY income_share_pct DESC;

20. CPM & EPMV percentiles (25/50/75) per club 
```sql
*/
WITH b AS (
  SELECT DISTINCT 'SC Braga' AS club,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY cpm)  OVER () AS cpm_p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY cpm)  OVER () AS cpm_p50,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY cpm)  OVER () AS cpm_p75,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY empv) OVER () AS epmv_p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY empv) OVER () AS epmv_p50,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY empv) OVER () AS epmv_p75
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT DISTINCT 'Sporting CP',
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY empv) OVER (),
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY empv) OVER (),
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY empv) OVER ()
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT DISTINCT N'Vitória SC',
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY cpm)  OVER (),
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY empv) OVER (),
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY empv) OVER (),
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY empv) OVER ()
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM b ORDER BY club;

21. Day-of-week revenue sums 
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club, DATENAME(WEEKDAY, [date]) AS weekday_name,
         SUM(daily_website_income) AS income
  FROM dbo.scbraga_pt
  GROUP BY DATENAME(WEEKDAY, [date])
  UNION ALL
  SELECT 'Sporting CP', DATENAME(WEEKDAY, [date]),
         SUM(daily_website_income)
  FROM dbo.sportingcp_pt
  GROUP BY DATENAME(WEEKDAY, [date])
  UNION ALL
  SELECT N'Vitória SC', DATENAME(WEEKDAY, [date]),
         SUM(daily_website_income)
  FROM dbo.vitoria_sc_pt
  GROUP BY DATENAME(WEEKDAY, [date])
)
SELECT * FROM t ORDER BY club, income DESC;

22. Weekend vs weekday performance 
```sql
*/
WITH t AS (
  SELECT 'SC Braga' AS club,
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) IN (1,7)
                  THEN daily_website_income ELSE 0 END) AS weekend_income,
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) NOT IN (1,7)
                  THEN daily_website_income ELSE 0 END) AS weekday_income
  FROM dbo.scbraga_pt
  UNION ALL
  SELECT 'Sporting CP',
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) IN (1,7)
                  THEN daily_website_income ELSE 0 END),
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) NOT IN (1,7)
                  THEN daily_website_income ELSE 0 END)
  FROM dbo.sportingcp_pt
  UNION ALL
  SELECT N'Vitória SC',
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) IN (1,7)
                  THEN daily_website_income ELSE 0 END),
         SUM(CASE WHEN DATEPART(WEEKDAY, [date]) NOT IN (1,7)
                  THEN daily_website_income ELSE 0 END)
  FROM dbo.vitoria_sc_pt
)
SELECT * FROM t;


23. Recomputed revenue per 1,000 (dates coerced to 2022, with your totals) 
```sql
*/
;WITH all_days AS (
    SELECT 'SC Braga'    AS club,
           DATEFROMPARTS(2022, MONTH([date]), DAY([date])) AS norm_date,
           daily_website_income, daily_unique_visits, daily_page_views
    FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP',
           DATEFROMPARTS(2022, MONTH([date]), DAY([date])),
           daily_website_income, daily_unique_visits, daily_page_views
    FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC',
           DATEFROMPARTS(2022, MONTH([date]), DAY([date])),
           daily_website_income, daily_unique_visits, daily_page_views
    FROM dbo.vitoria_sc_pt
),
agg AS (
    SELECT club,
           MIN(norm_date) AS first_date_2022,
           MAX(norm_date) AS last_date_2022,
           SUM(CAST(daily_website_income AS decimal(18,6))) AS inc,
           SUM(CAST(daily_unique_visits  AS bigint))        AS uv,
           SUM(CAST(daily_page_views     AS bigint))        AS pv
    FROM all_days
    GROUP BY club
),
provided AS (
    SELECT * FROM (VALUES
        ('SC Braga',    CAST(163535.00 AS decimal(18,2))),
        ('Sporting CP', CAST(185125.00 AS decimal(18,2))),
        ('Vitoria SC',  CAST(155120.00 AS decimal(18,2)))
    ) v(club, provided_total_revenue)
)
SELECT
    a.club,
    a.first_date_2022,
    a.last_date_2022,
    -- weighted metrics from your data
    1000.0 * a.inc / NULLIF(a.uv,0) AS epmv_weighted,
    1000.0 * a.inc / NULLIF(a.pv,0) AS cpm_weighted,
    -- your provided totals (for quick comparison)
    p.provided_total_revenue
FROM agg a
LEFT JOIN provided p ON p.club = a.club
ORDER BY epmv_weighted DESC, cpm_weighted DESC;


24. Outlier days grouped into 2021 vs 2022 buckets (per club) 
```sql
*/
;WITH all_days AS (
    SELECT 'SC Braga'    AS club, [date], daily_website_income FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], daily_website_income FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC'  AS club, [date], daily_website_income FROM dbo.vitoria_sc_pt
),
norm AS (
    -- Map dates: <= 2021 -> 2021,  > 2021 -> 2022
    SELECT
        club,
        CASE WHEN YEAR([date]) <= 2021 THEN 2021 ELSE 2022 END AS norm_year,
        CAST(daily_website_income AS float) AS income
    FROM all_days
),
stats AS (
    -- Mean & stdev per club (across all rows for that club)
    SELECT club,
           AVG(income)  AS mu,
           STDEV(income) AS sigma
    FROM norm
    GROUP BY club
),
tag AS (
    SELECT n.club, n.norm_year, n.income,
           s.mu, s.sigma,
           CASE WHEN n.income > s.mu + 2*s.sigma THEN 1 ELSE 0 END AS is_outlier
    FROM norm n
    JOIN stats s ON s.club = n.club
)
SELECT
    club,
    norm_year AS [year],
    SUM(CASE WHEN is_outlier = 1 THEN 1 ELSE 0 END)                         AS outlier_days,
    SUM(CASE WHEN is_outlier = 1 THEN income ELSE 0 END)                     AS outlier_income_total,
    AVG(CASE WHEN is_outlier = 1 THEN income END)                            AS avg_outlier_income,
    MAX(CASE WHEN is_outlier = 1 THEN income END)                            AS max_outlier_income
FROM tag
GROUP BY club, norm_year
ORDER BY club, [year];


25. Final overall ranking (override totals, normalized 2021–2022, cost = 125,000)
  
Your totals:
   - SC Braga     = 175,235.00
   - Sporting CP  = 192,250.00
   - Vitoria SC   = 155,220.00
```sql
*/
;WITH d AS (
    SELECT 'SC Braga'    AS club, [date], daily_page_views, daily_unique_visits, daily_website_income FROM dbo.scbraga_pt
    UNION ALL
    SELECT 'Sporting CP' AS club, [date], daily_page_views, daily_unique_visits, daily_website_income FROM dbo.sportingcp_pt
    UNION ALL
    SELECT 'Vitoria SC'  AS club, [date], daily_page_views, daily_unique_visits, daily_website_income FROM dbo.vitoria_sc_pt
),
normalized AS (
    -- Force Aug–Dec to 2021; Jan–Jun to 2022; keep only Aug..Jun
    SELECT
        club,
        2022 AS season_year,
        DATEFROMPARTS(CASE WHEN MONTH([date]) >= 8 THEN 2021 ELSE 2022 END,
                      MONTH([date]), DAY([date])) AS season_date,
        daily_page_views, daily_unique_visits, daily_website_income
    FROM d
    WHERE MONTH([date]) IN (8,9,10,11,12,1,2,3,4,5,6)
),
agg AS (
    -- Keep tie-breaker metrics from data
    SELECT
        club,
        season_year,
        SUM(daily_page_views)     AS pv,
        SUM(daily_unique_visits)  AS uv,
        SUM(daily_website_income) AS inc
    FROM normalized
    GROUP BY club, season_year
),
provided AS (
    -- Override totals you supplied (ROI will use these)
    SELECT * FROM (VALUES
        ('SC Braga',    2022, CAST(175235.00 AS decimal(18,2))),
        ('Sporting CP', 2022, CAST(192250.00 AS decimal(18,2))),
        ('Vitoria SC',  2022, CAST(155220.00 AS decimal(18,2)))
    ) v(club, season_year, season_revenue_total)
)
SELECT
    a.club,
    a.season_year,
    CAST(p.season_revenue_total AS money) AS season_revenue_total,
    CAST(125000 AS money)                 AS sponsor_cost,
    -- ROI as positive percent + outcome
    CAST(ROUND(ABS((p.season_revenue_total - 125000) * 100.0 / 125000.0), 2) AS decimal(8,2)) AS roi_percent,
    CASE WHEN p.season_revenue_total >= 125000 THEN 'Profit' ELSE 'Shortfall' END AS outcome,
    1000.0 * a.inc / NULLIF(a.pv,0) AS weighted_cpm,
    1000.0 * a.inc / NULLIF(a.uv,0) AS weighted_epmv
FROM agg a
JOIN provided p
  ON p.club = a.club AND p.season_year = a.season_year
ORDER BY roi_percent DESC, weighted_epmv DESC, weighted_cpm DESC;

1. Season headline: total attendance, avg per match, avg occupancy (fixed column names) 
```sql
*/
;WITH events AS (
    SELECT 'SC Braga' AS club,
           TRY_CONVERT(int, [match_number])           AS match_no,
           TRY_CONVERT(int, [attendance])             AS att,
           TRY_CONVERT(int, [average_attendance])     AS avg_att,
           COALESCE(
               TRY_CONVERT(decimal(6,2), [occupancy]),
               TRY_CONVERT(decimal(6,2), REPLACE(CONVERT(varchar(32), [occupancy]), ',', '.'))
           )                                          AS occ_pct
    FROM dbo.scbraga_reports

    UNION ALL
    SELECT 'Sporting CP',
           TRY_CONVERT(int, [match_number]),
           TRY_CONVERT(int, [attendance]),
           TRY_CONVERT(int, [average_attendance]),
           COALESCE(
               TRY_CONVERT(decimal(6,2), [occupancy]),
               TRY_CONVERT(decimal(6,2), REPLACE(CONVERT(varchar(32), [occupancy]), ',', '.'))
           )
    FROM dbo.sportingcp_reports

    UNION ALL
    SELECT 'Vitoria SC',
           TRY_CONVERT(int, [match_number]),
           TRY_CONVERT(int, [attendance]),
           TRY_CONVERT(int, [average_attendance]),
           COALESCE(
               TRY_CONVERT(decimal(6,2), [occupancy]),
               TRY_CONVERT(decimal(6,2), REPLACE(CONVERT(varchar(32), [occupancy]), ',', '.'))
           )
    FROM dbo.vitoriasc_reports
)
SELECT
    '2021-2022'                           AS season_label,
    club,
    SUM(att)                              AS total_attendance,
    AVG(CAST(att AS decimal(18,2)))       AS avg_attendance_per_match,
    AVG(occ_pct)                          AS avg_occupancy_pct
FROM events
GROUP BY club
ORDER BY total_attendance DESC;


2. Who wins most events? (by attendance, count of match wins) 
```sql
*/
;WITH events AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[match_number]) AS match_no, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL
  SELECT 'Sporting CP', TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL
  SELECT 'Vitoria SC',  TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
),
ranked AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY match_no ORDER BY att DESC) AS rn
  FROM events
)
SELECT club, COUNT(*) AS match_wins
FROM ranked
WHERE rn = 1
GROUP BY club
ORDER BY match_wins DESC;


3. engagement per event: avg occupancy + “sell-out” count (>= 90%) 
```sql
*/
;WITH events AS (
  SELECT 'SC Braga' AS club,
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.'))) AS occ_pct
  FROM dbo.scbraga_reports
  UNION ALL
  SELECT 'Sporting CP',
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.')))
  FROM dbo.sportingcp_reports
  UNION ALL
  SELECT 'Vitoria SC',
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.')))
  FROM dbo.vitoriasc_reports
)
SELECT club,
       AVG(occ_pct)                                   AS avg_occupancy_pct,
       SUM(CASE WHEN occ_pct >= 90 THEN 1 ELSE 0 END) AS sellout_like_events
FROM events
GROUP BY club
ORDER BY avg_occupancy_pct DESC, sellout_like_events DESC;


4. Consistency: volatility & coefficient of variation (lower = steadier) 
```sql
*/
;WITH events AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL SELECT 'Sporting CP', TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL SELECT 'Vitoria SC',  TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
)
SELECT club,
       AVG(CAST(att AS float)) AS mean_attendance,
       STDEV(CAST(att AS float)) AS std_attendance,
       STDEV(CAST(att AS float)) / NULLIF(AVG(CAST(att AS float)),0) AS coeff_variation
FROM events
GROUP BY club
ORDER BY coeff_variation ASC;

5. Top-3 matches by attendance per club 
```sql
*/
;WITH e AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[match_number]) AS match_no, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL SELECT 'Sporting CP', TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL SELECT 'Vitoria SC',  TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
),
r AS (
  SELECT e.*, ROW_NUMBER() OVER (PARTITION BY club ORDER BY att DESC) AS rn FROM e
)
SELECT club, match_no, att
FROM r
WHERE rn <= 3
ORDER BY club, att DESC;


6. Median & 90th-percentile attendance per club 
```sql
*/
;WITH e AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL SELECT 'Sporting CP', TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL SELECT 'Vitoria SC',  TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
)
SELECT DISTINCT
  club,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY att) OVER (PARTITION BY club) AS median_attendance,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY att) OVER (PARTITION BY club) AS p90_attendance
FROM e
ORDER BY club;


7. Over/under performance vs each club’s own average 
```sql
*/
;WITH e AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[match_number]) AS match_no, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL SELECT 'Sporting CP', TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL SELECT 'Vitoria SC',  TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
),
w AS (
  SELECT e.*, AVG(CAST(att AS float)) OVER (PARTITION BY club) AS club_avg FROM e
)
SELECT club,
       SUM(CASE WHEN att >= club_avg THEN 1 ELSE 0 END)          AS events_above_own_avg,
       AVG(CASE WHEN att >= club_avg THEN att - club_avg END)     AS avg_lift_when_above
FROM w
GROUP BY club
ORDER BY events_above_own_avg DESC, avg_lift_when_above DESC;


8. Match-by-match table: 1st/2nd/3rd by attendance each round 
```sql
*/
;WITH e AS (
  SELECT 'SC Braga' AS club, TRY_CONVERT(int,[match_number]) AS match_no, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
  UNION ALL SELECT 'Sporting CP', TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.sportingcp_reports
  UNION ALL SELECT 'Vitoria SC',  TRY_CONVERT(int,[match_number]), TRY_CONVERT(int,[attendance]) FROM dbo.vitoriasc_reports
),
r AS (
  SELECT e.*, DENSE_RANK() OVER (PARTITION BY match_no ORDER BY att DESC) AS pos FROM e
)
SELECT match_no,
       MAX(CASE WHEN pos=1 THEN club + ' ('+CONVERT(varchar(10),att)+')' END) AS winner,
       MAX(CASE WHEN pos=2 THEN club + ' ('+CONVERT(varchar(10),att)+')' END) AS second,
       MAX(CASE WHEN pos=3 THEN club + ' ('+CONVERT(varchar(10),att)+')' END) AS third
FROM r
GROUP BY match_no
ORDER BY match_no;


9. Revenue potential ranking (club-specific ticket prices) 
```sql
*/
;WITH e AS (
    SELECT 'SC Braga'    AS club, TRY_CONVERT(int,[attendance]) AS att FROM dbo.scbraga_reports
    UNION ALL
    SELECT 'Sporting CP' AS club, TRY_CONVERT(int,[attendance])       FROM dbo.sportingcp_reports
    UNION ALL
    SELECT 'Vitoria SC'  AS club, TRY_CONVERT(int,[attendance])       FROM dbo.vitoriasc_reports
),
price AS (
    SELECT * FROM (VALUES
        ('SC Braga',    CAST(23.00 AS decimal(10,2))),
        ('Sporting CP', CAST(30.00 AS decimal(10,2))),
        ('Vitoria SC',  CAST(17.00 AS decimal(10,2)))
    ) p(club, avg_ticket_price)
),
agg AS (
    SELECT
        e.club,
        COUNT(*)                                   AS events,
        SUM(e.att)                                 AS total_attendance,
        p.avg_ticket_price,
        AVG(CAST(e.att AS decimal(18,2)))          AS avg_attendance_per_event,
        SUM(e.att) * p.avg_ticket_price            AS gross_revenue_raw,
        p.avg_ticket_price * AVG(CAST(e.att AS decimal(18,2))) AS avg_revenue_per_event_raw
    FROM e
    JOIN price p ON p.club = e.club
    GROUP BY e.club, p.avg_ticket_price
)
SELECT
    club,
    events,
    total_attendance,
    avg_ticket_price,
    CAST(gross_revenue_raw       AS money)          AS gross_ticket_revenue,
    CAST(avg_attendance_per_event AS decimal(18,2)) AS avg_attendance_per_event,
    CAST(avg_revenue_per_event_raw AS money)        AS avg_revenue_per_event,
    CAST(ROUND(100.0 * gross_revenue_raw
               / NULLIF(SUM(gross_revenue_raw) OVER (),0), 2) AS decimal(6,2)) AS revenue_share_pct
FROM agg
ORDER BY gross_ticket_revenue DESC;


10. Final scoreboard (attendance share + occupancy + sell-outs) 
```sql
*/
;WITH e AS (
  SELECT 'SC Braga' AS club,
         TRY_CONVERT(int,[attendance]) AS att,
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.'))) AS occ_pct
  FROM dbo.scbraga_reports
  UNION ALL
  SELECT 'Sporting CP',
         TRY_CONVERT(int,[attendance]),
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.')))
  FROM dbo.sportingcp_reports
  UNION ALL
  SELECT 'Vitoria SC',
         TRY_CONVERT(int,[attendance]),
         COALESCE(TRY_CONVERT(decimal(6,2),[occupancy]),
                  TRY_CONVERT(decimal(6,2),REPLACE(CONVERT(varchar(32),[occupancy]),',','.')))
  FROM dbo.vitoriasc_reports
),
club AS (
  SELECT club,
         SUM(att) AS total_att,
         AVG(occ_pct) AS avg_occ,
         SUM(CASE WHEN occ_pct >= 90 THEN 1 ELSE 0 END) AS sellouts
  FROM e
  GROUP BY club
),
tot AS (
  SELECT SUM(total_att) AS all_att, SUM(sellouts) AS all_sellouts FROM club
),
score AS (
  SELECT c.*,
         1.0 * c.total_att / NULLIF(t.all_att,0)      AS att_share,
         1.0 * c.sellouts  / NULLIF(t.all_sellouts,0) AS sellout_share
  FROM club c CROSS JOIN tot t
)
SELECT club,
       total_att,
       avg_occ                   AS avg_occupancy_pct,
       sellouts                  AS sellout_like_events,
       (0.50*att_share + 0.30*(avg_occ/100.0) + 0.20*sellout_share) AS composite_score
FROM score
ORDER BY composite_score DESC;
