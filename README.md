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
