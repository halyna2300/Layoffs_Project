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

