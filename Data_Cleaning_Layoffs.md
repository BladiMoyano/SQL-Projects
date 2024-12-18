# SQL-Projects

**DATA CLEANING PROCESS**
1. Remove Duplicates
2. Standardize Data
3. Take care of Null/Blank Values
4. Remove Unnecessary Columns


## Creating a working copy of the data

```sql
CREATE TABLE layoffs_staging
AS 
SELECT * FROM layoffs;
```

## 1. Removing Duplicates

```sql
SELECT *,
ROW_NUMBER( ) OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

WITH delete_cte AS
(
SELECT *,
ROW_NUMBER( ) OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM delete_cte
WHERE row_num > 1;

-- Creating a secondary table without duplicates

CREATE TABLE `layoffs_staging_2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT layoffs_staging_2
SELECT *,
ROW_NUMBER( ) OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

SELECT *
FROM layoffs_staging_2

```

## 2 Standarding data

```sql
SELECT company, TRIM(company)
FROM layoffs_staging_2;

UPDATE layoffs_staging_2
SET company = TRIM(company);
---
SELECT DISTINCT industry
FROM layoffs_staging_2
ORDER BY 1;

UPDATE layoffs_staging_2
SET industry = "Crypto"
WHERE industry like "%Crypto%";

SELECT DISTINCT industry
FROM layoffs_staging_2;
---
SELECT DISTINCT location
FROM layoffs_staging_2
ORDER BY 1;
---
SELECT DISTINCT country
FROM layoffs_staging_2
ORDER BY 1;
UPDATE layoffs_staging_2
SET country = "United States"
WHERE country like "%United States%";
---
SELECT `date`, str_to_date(`date`, "%m/%d/%Y")
FROM layoffs_staging_2;

UPDATE layoffs_staging_2
SET `date`= str_to_date(`date`, "%m/%d/%Y");

SELECT *
FROM layoffs_staging_2;

ALTER TABLE layoffs_staging_2
MODIFY COLUMN `date` DATE;
```

## 3. Populating Nulls and Blanks

```sql
SELECT *
FROM layoffs_staging_2
WHERE industry IS NULL
OR industry = "";

UPDATE layoffs_staging_2
SET industry = NULL
WHERE industry = "";

SELECT t1.company, t1.location, t1.industry, t2.company, t2.location, t2.industry
FROM layoffs_staging_2 t1
JOIN layoffs_staging_2 t2
	ON t1.company = t2.company
    AND t1. location = t2.location
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging_2 t1
JOIN layoffs_staging_2 t2
	ON t1.company = t2.company
    AND t1. location = t2.location
SET t1.industry = t2.industry 
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

## 4. Removing Unnecessary Columns

```sql
ALTER TABLE layoffs_staging_2
DROP COLUMN row_num;

SELECT *
FROM layoffs_staging_2;
```

