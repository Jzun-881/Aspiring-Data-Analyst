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

## 👨‍💻 Author

**Manjunatha B N**

---

