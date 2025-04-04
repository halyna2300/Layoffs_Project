<h1 align="center">Layoffs Data Cleaning & Analysis Project</h1>

https://github.com/halyna2300/Layoffs_Project/blob/main/Layoffs.jpg

## Overview
This project focuses on cleaning and analyzing a dataset containing global layoffs data. The dataset was sourced from Kaggle. The objective is to perform data cleaning, standardization, and exploratory data analysis (EDA) using SQL.

## Objectives
- Clean and standardize raw layoffs data for better usability.
- Identify trends and patterns in layoffs across different industries and time periods.
- Explore key factors such as funding, location, and industry impact.
- Develop SQL queries to extract meaningful insights from the dataset.
  
## Dataset
The data for this project is sourced from the Kaggle dataset:
- **Dataset Link:** [Layoffs Dataset](https://www.kaggle.com/datasets/swaptr/layoffs-2022)


## Schema
```sql
CREATE TABLE `layoffs_staging` ( 
    `company` text,
    `location` text, 
    `industry` text,
    `total_laid_off` int DEFAULT NULL, 
    `percentage_laid_off` text,
    `date` text, 
    `stage` text,
    `country` text,
    `funds_raised_millions` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

## Data Cleaning Steps
- ** Remove duplicates**
- ** Standardize the Data**
- ** Null values/ blank values**
- ** Remove any columns**

-- Checking the initial dataset before applying changes
```sql
 SELECT * FROM layoffs;
```

-- Creating a staging table to work in and keep original data safe
```sql
CREATE TABLE layoffs_staging LIKE layoffs;
```

```sql
SELECT * FROM layoffs_staging;
```

```sql
INSERT INTO layoffs_staging SELECT * FROM layoffs;
```

-- Identifying duplicate records using ROW_NUMBER() function
```sql
SELECT *,
ROW_NUMBER() OVER (
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```

-- Finding and displaying duplicate records
```sql
WITH duplicate_cte AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num FROM layoffs_staging
)
SELECT * FROM duplicate_cte WHERE row_num > 1;
```

-- Checking for duplicates in a specific company
```sql
SELECT * FROM layoffs_staging WHERE company = 'Oda';
```

-- Removing duplicate records from the dataset
```sql
WITH duplicate_cte AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num FROM layoffs_staging
)
DELETE FROM duplicate_cte WHERE row_num > 1;
```

-- Creating a new staging table with an additional column for row numbers
```sql
DROP TABLE IF EXISTS layoffs_staging2;
CREATE TABLE layoffs_staging2 (
    `company` text,
    `location` text,
    `industry` text,
    `total_laid_off` text,
    `percentage_laid_off` text,
    `date` text,
    `stage` text,
    `country` text,
    `funds_raised_millions` text,
    `row_num` text
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

-- Inserting data into new staging table and assigning row numbers
```sql
INSERT INTO layoffs_staging2
SELECT *, ROW_NUMBER() OVER (
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
) AS row_num FROM layoffs_staging;
```

```sql
SELECT * FROM layoffs_staging2;
```
-- Updating empty fields to NULL for easier handling
```sql
UPDATE layoffs_staging2 SET date = NULL WHERE date = ' ';
UPDATE layoffs_staging2 SET total_laid_off = NULL WHERE total_laid_off = '';
UPDATE layoffs_staging2 SET funds_raised_millions = NULL WHERE funds_raised_millions = '' OR funds_raised_millions = 'None';
```
-- Standardizing column values to maintain consistency
```sql
UPDATE layoffs_staging2 SET company = TRIM(company);
UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry LIKE 'Crypto%';
UPDATE layoffs_staging2 SET country = TRIM(TRAILING '.' FROM country) WHERE country LIKE 'United States%';
UPDATE layoffs_staging2 SET date = STR_TO_DATE(date, '%m/%d/%Y');
```

-- Changing the data type of the date column
```sql
ALTER TABLE layoffs_staging2 MODIFY COLUMN `date` DATE;
```
-- Removing rows with no meaningful data
```sql
DELETE FROM layoffs_staging2 WHERE total_laid_off IS NULL
AND percentage_laid_off = 'None';
```
-- Dropping the temporary row number column
```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```
## Exploratory Data Analysis
```sql
SELECT *
FROM layoffs_staging2;
```
-- Finding the maximum layoffs and highest percentage of layoffs
```sql
SELECT MAX(total_laid_off),MAX(percentage_laid_off)
FROM layoffs_staging2;
```

-- Identifying companies with 100% layoffs
```sql
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```

```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```

```sgl
SELECT MIN(date),MAX(date)
FROM layoffs_staging2;
```

-- Ranking industries and companies by the number of layoffs
```sql
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```

```sql
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 1 DESC;
```

-- Analyzing layoffs trends by year
```sql
SELECT YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 DESC;
```

```sql
SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;
```

```sql
SELECT company, SUM(percentage_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```

-- Analyzing layoffs per month to identify trends over time
```sql
SELECT SUBSTRING(`date`, 1,7) AS `MONTH`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`, 1,7) IS NOT NULL
GROUP BY `MONTH` 
ORDER BY 1 ASC;
```

-- Calculating rolling total layoffs over time to visualize trends
```sql
WITH Rolling_Total AS
 (
 SELECT SUBSTRING(`date`, 1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
WHERE SUBSTRING(`date`, 1,7) IS NOT NULL
GROUP BY `MONTH` 
ORDER BY 1 ASC
)
SELECT `MONTH`,total_off, 
SUM(total_off)OVER(ORDER BY `MONTH`) AS rolling_total
FROM Rolling_Total;
```

```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```

-- Aggregating layoffs by company and year to track layoffs over time
```sql
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(`date`)
ORDER BY 3 DESC;
```

-- Ranking companies with the highest layoffs per year using DENSE_RANK()
```sql
WITH Company_Year (company,years,total_laid_off) AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(`date`)
)
SELECT*,DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
ORDER BY ranking ASC;
```

-- Identifying the top 5 companies with the highest layoffs per year
```sql
WITH Company_Year (company,years,total_laid_off) AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company,YEAR(`date`)
),Company_Year_Rank AS
(SELECT*,DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE ranking <=5
;
```
