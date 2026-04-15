# Aspiring-Data-Analyst

# 📊 Data Cleaning in MySQL: Real-World Layoffs Dataset

## 🚀 Project Overview

This project demonstrates how to clean messy, real-world data using MySQL.
Using a global layoffs dataset, the goal is to transform raw, inconsistent data into a structured, analysis-ready format.

> ⚠️ In real-world data work, cleaning is not optional—it’s where the actual value is created.

---

## 🧠 Key Learning Objectives

* Handle missing and inconsistent data
* Remove duplicates without a primary key
* Standardize categorical values
* Convert text-based fields into proper data types
* Use self-joins to recover missing values
* Apply production-level data cleaning practices

---

## 📂 Dataset

* **Name:** World Layoffs Dataset
* **Description:** Contains global layoff events across companies, industries, and regions
* **Fields Include:**

  * Company
  * Location
  * Industry
  * Total Laid Off
  * Percentage Laid Off
  * Date
  * Stage
  * Country
  * Funds Raised

---

## ⚙️ Project Workflow

### 1. Create a Staging Table (Protect Raw Data)

Raw data should never be modified directly.

```sql
CREATE TABLE layoffs_staging LIKE layoffs;

INSERT INTO layoffs_staging
SELECT * FROM layoffs;
```

✅ Ensures:

* Backup safety
* Pipeline stability
* Reusability

---

### 2. Remove Duplicates

Since there is no primary key, duplicates are identified using all columns.

```sql
SELECT *,
ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off,
    percentage_laid_off, date, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;
```

⚠️ MySQL Limitation:

* Cannot delete directly from a CTE

✔️ Solution:

* Create a second staging table (`layoffs_staging2`)
* Store `row_num`
* Delete where `row_num > 1`

---

### 3. Standardize Data

#### Fix inconsistent values

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

#### Remove unwanted characters

```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);
```

---

### 4. Convert Data Types

Dates imported as text must be converted:

```sql
UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
```

✅ Enables time-series analysis

---

### 5. Handle Missing Values

#### Convert blanks to NULL

```sql
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
```

#### Use self-join to fill missing data

```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
  ON t1.company = t2.company
 AND t1.location = t2.location
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
  AND t2.industry IS NOT NULL;
```

---

### 6. Remove Irrelevant Rows

Delete rows that provide no analytical value:

```sql
DELETE FROM layoffs_staging2
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;
```

---

### 7. Final Cleanup

Remove helper columns:

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## 📈 Before vs After (Example)

| Stage        | Rows | Notes                    |
| ------------ | ---- | ------------------------ |
| Raw Data     | XXXX | Duplicates, nulls, messy |
| Cleaned Data | XXXX | Analysis-ready dataset   |

*(Update this with actual numbers if available)*

---

## 💡 Key Takeaways

* Raw data should always remain untouched
* Duplicate detection requires full-row comparison
* MySQL has limitations that require workarounds
* Data cleaning is iterative and non-linear
* Self-joins are powerful for recovering missing data
* Not all data should be kept—some must be removed

---

## 🧪 Skills Demonstrated

* SQL (MySQL)
* Data Cleaning & Preprocessing
* Window Functions
* Joins & Self-Joins
* Data Standardization
* Problem Solving with Real Data

---

## 🔥 What’s Next?

* Perform Exploratory Data Analysis (EDA)
* Build visualizations (Tableau / Power BI / Python)
* Extract business insights from layoffs trends

---

## 📌 Final Thought

> Clean data is not a luxury—it’s a requirement.

Messy data leads to misleading insights.
Well-cleaned data leads to confident decisions.

---

---

# 📊 Exploratory Data Analysis: Tech Layoffs (2020–2023)

## 🔍 Project Objective

After cleaning the dataset, this phase focuses on uncovering **real-world insights** behind global tech layoffs.

Instead of relying on headlines, this analysis uses SQL-based EDA techniques such as:

* Common Table Expressions (CTEs)
* Rolling totals
* Aggregations
* Ranking functions

👉 Goal: Identify patterns, trends, and structural shifts in the tech industry.

---

## 📅 Dataset Scope

* **Time Period:** March 2020 – March 2023
* **Total Layoffs Analyzed:** ~383,000
* **Global Coverage:** Multiple countries and industries

---

## 📈 Key Insights

---

### 🚨 1. Layoffs Exploded in Early 2023

The most shocking trend is the **speed of layoffs in 2023**.

* **2022 Total:** ~160,000 layoffs
* **First 3 months of 2023:** ~125,000 layoffs

👉 In just 90 days, 2023 nearly matched the entire previous year.

**Insight:**
This is not a slow decline—it’s a **rapid contraction phase**.

---

### 🧊 2. 2021 Was an Artificial “Calm Period”

* **2020:** ~80,000 layoffs (pandemic shock)
* **2021:** ~15,000 layoffs (sharp drop)

Despite global uncertainty, layoffs temporarily decreased.

👉 This created a **false sense of stability**.

**Insight:**
2021 delayed the correction that exploded in 2022–2023.

---

### 🏢 3. Big Tech Was Hit Hardest (Not Startups)

Contrary to popular belief:

* Largest layoffs came from **Post-IPO companies**
* Not early-stage startups

Major companies impacted include:

* Amazon
* Google
* Meta

👉 Example: A single **12,000 employee layoff** by Google.

**Insight:**
This was not startup failure—it was **industry-wide restructuring**.

---

### 🌍 4. U.S. and Consumer Sector Took the Biggest Hit

#### 🌎 By Geography:

* 🇺🇸 United States → ~256,000 layoffs (highest by far)
* Followed by India, Netherlands

#### 🏬 By Industry:

* Consumer & Retail → most affected
* Manufacturing, Energy → relatively stable

**Insight:**
Customer-facing industries were the most vulnerable during disruptions.

---

### 💀 5. Some Companies Didn’t Survive at All

Filtering for:

```sql
percentage_laid_off = 1
```

👉 Means **100% workforce laid off**

Examples:

* Quibi
* BlockFi
* Volt Bank

**Insight:**
Even heavily funded companies can collapse completely.

---

## 🧠 SQL Techniques Used

### 📌 Rolling Total (Layoff Trend Over Time)

```sql
WITH monthly_layoffs AS (
    SELECT 
        DATE_FORMAT(date, '%Y-%m') AS month,
        SUM(total_laid_off) AS layoffs
    FROM layoffs_staging2
    GROUP BY month
)
SELECT 
    month,
    SUM(layoffs) OVER (ORDER BY month) AS rolling_total
FROM monthly_layoffs;
```

---

### 📌 Top Companies by Layoffs (Per Year)

```sql
SELECT *
FROM (
    SELECT 
        company,
        YEAR(date) AS year,
        SUM(total_laid_off) AS total,
        DENSE_RANK() OVER (
            PARTITION BY YEAR(date)
            ORDER BY SUM(total_laid_off) DESC
        ) AS rank_num
    FROM layoffs_staging2
    GROUP BY company, year
) ranked
WHERE rank_num <= 5;
```

---

## 📊 Key Takeaways

* Layoffs are **accelerating, not stabilizing**
* 2021 masked underlying instability
* Large corporations drove the majority of job cuts
* Economic impact is concentrated geographically and sector-wise
* Some companies experienced **complete shutdowns**

---

## ⚠️ Limitations

* Dataset includes **reported layoffs only**
* Likely excludes:

  * Small businesses
  * Unreported job losses

👉 Actual impact may be significantly higher

---

## 🔮 Final Insight

> The data suggests this is not a temporary disruption—but a structural shift.

The tech industry is moving toward:

* Leaner operations
* Cost optimization
* Reduced over-hiring

---

## 🚀 Next Steps

* Build dashboards (Power BI / Tableau)
* Perform time-series forecasting
* Compare layoffs vs funding trends
* Add visualizations using Python (Matplotlib / Seaborn)

---

👨‍💻 Author

Manjunatha B N

---
