# Layoffs Data Cleaning & Analysis Project

![Layoffs Project Image](https://github.com/halyna2300/Layoffs_Project/blob/main/IMG_8560.JPG)

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
-- Remove duplicates;
-- Standardize the Data;
-- Null values/ blank values;
-- Remove any columns;

## SELECT * FROM layoffs;

```sql
CREATE TABLE layoffs_staging LIKE layoffs;

SELECT * FROM layoffs_staging;

INSERT INTO layoffs_staging SELECT * FROM layoffs;
```

```sql
SELECT *,
ROW_NUMBER() OVER (
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```


```sql
WITH duplicate_cte AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num FROM layoffs_staging
)
SELECT * FROM duplicate_cte WHERE row_num > 1;
```

```sql
SELECT * FROM layoffs_staging WHERE company = 'Oda';
```


```sql
WITH duplicate_cte AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num FROM layoffs_staging
)
DELETE FROM duplicate_cte WHERE row_num > 1;
```


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

INSERT INTO layoffs_staging2
SELECT *, ROW_NUMBER() OVER (
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
) AS row_num FROM layoffs_staging;

SELECT * FROM layoffs_staging2;

UPDATE layoffs_staging2 SET date = NULL WHERE date = ' ';
UPDATE layoffs_staging2 SET total_laid_off = NULL WHERE total_laid_off = '';
UPDATE layoffs_staging2 SET funds_raised_millions = NULL WHERE funds_raised_millions = '' OR funds_raised_millions = 'None';

UPDATE layoffs_staging2 SET company = TRIM(company);
UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry LIKE 'Crypto%';
UPDATE layoffs_staging2 SET country = TRIM(TRAILING '.' FROM country) WHERE country LIKE 'United States%';
UPDATE layoffs_staging2 SET date = STR_TO_DATE(date, '%m/%d/%Y');
ALTER TABLE layoffs_staging2 MODIFY COLUMN `date` DATE;

DELETE FROM layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off = 'None';
ALTER TABLE layoffs_staging2 DROP COLUMN row_num;
