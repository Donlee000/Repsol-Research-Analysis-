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
