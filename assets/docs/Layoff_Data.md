# SQL-Project.
Data Cleaning & Exploration of Layoffs in 2020, 2021, and 2022.

Here you can find the raw dataset.

## Data Cleaning
1. Remove Duplicates
2. Standardize Data
3. Handling of Null/Blank Values
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

## 2 Standardizing data

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

##  Layoffs Overview
### Top 5 industries with the most layoffs ###
```sql
SELECT industry, sum(total_laid_off) as top_layoffs
FROM layoffs_staging_2
WHERE industry != "other"
GROUP BY industry
ORDER BY top_layoffs desc
LIMIT 5;
```

![Querry-1](/assets/images/querry-1.PNG)

### Top 5 Companies with the Most Layoffs Per Year ###

```sql 
WITH t1 as (
  SELECT company, total_laid_off, YEAR(`date`) as years
  FROM layoffs_staging_2
  WHERE total_laid_off IS NOT NULL
  AND YEAR(`date`) IS NOT NULL),
t2 as(
  SELECT *, DENSE_RANK() OVER(partition by years order by total_laid_off DESC) as rank_layoff
  FROM t1
)
SELECT *
FROM t2
WHERE rank_layoff <= 5;
```

![Querry-2](/assets/images/querry-2.PNG)


## Monthly Trend of Layoffs ##

```sql
SELECT sum(total_laid_off), MONTH(`date`) as layoff_month, YEAR(`date`) as layoff_year
FROM layoffs_staging_2
where `date` is not null
group by layoff_month, layoff_year
order by layoff_year, layoff_month;
```

![Querry-3](/assets/images/querry-3.PNG)


## Location Analysis ##
### Countries with the Most Layoffs Outside the United States ###

```sql
WITH t1 as(
  SELECT country, SUM(total_laid_off) as layoffs_country
  FROM layoffs_staging_2
  WHERE country != "United States"
  GROUP BY country
  ORDER BY layoffs_country DESC),
t2 as (
  SELECT *, rank() over(order by layoffs_country DESC) as rank_countries
  FROM t1)
SELECT *
FROM t2
WHERE rank_countries <= 5;
```

![Querry-4](/assets/images/querry-4.PNG)

### 10 American cities with the most layoffs ###

```sql
select location, sum(total_laid_off)
from layoffs_staging_2
where country = 'United States'
group by location
order by sum(total_laid_off) DESC
limit 10;
```

![Querry-5](/assets/images/querry-5.1.PNG)

## Funding Analysis ##
### Relationship Between Funds Raised and Percentage of Layoffs ###
#### Average Funds for Companies with 0-50% Layoffs ####

```sql
SELECT AVG(funds_raised_millions) AS avg_funds_raised_millions
FROM layoffs_staging_2
WHERE total_laid_off IS NOT NULL 
  AND percentage_laid_off IS NOT NULL 
  AND date IS NOT NULL 
  AND funds_raised_millions IS NOT NULL
  AND percentage_laid_off > 0
  AND percentage_laid_off < 0.5;
 ```
![Querry-4](/assets/images/querry-6.PNG)

#### Average Funds for Companies with 50-99% Layoffs ####
```sql
SELECT AVG(funds_raised_millions) AS avg_funds_raised_millions
FROM layoffs_staging_2
WHERE total_laid_off IS NOT NULL 
  AND percentage_laid_off IS NOT NULL 
  AND date IS NOT NULL 
  AND funds_raised_millions IS NOT NULL
  AND percentage_laid_off > 0.5
  AND percentage_laid_off < 0.99;
```
![Querry-4](/assets/images/querry-7.PNG)

#### Average Funds for Companies with 100% Layoffs (Total Closure) ####

```sql  
  SELECT percentage_laid_off, AVG(funds_raised_millions) AS avg_funds_raised_millions
FROM layoffs_staging_2
WHERE total_laid_off IS NOT NULL 
  AND percentage_laid_off IS NOT NULL 
  AND date IS NOT NULL 
  AND funds_raised_millions IS NOT NULL
  AND percentage_laid_off = 1;
```
![Querry-4](/assets/images/querry-8.PNG)

### Companies That Raised Significant Funds but Still Experienced Layoffs. ###

```sql
SELECT company, sum(funds_raised_millions) as funds, sum(total_laid_off) as total_layoff, CAST(AVG(percentage_laid_off) AS DECIMAL (10,2)) as percentage_layoff
FROM layoffs_staging_2
group by company
order by funds DESC
limit 10;
```

![Querry-4](/assets/images/querry-9.PNG)

### Companies with High Funding and High Layoffs ###

```sql
SELECT company, funds_raised_millions, total_laid_off
FROM layoffs_staging_2
WHERE funds_raised_millions > (SELECT AVG(funds_raised_millions) FROM layoffs_staging_2)
  AND total_laid_off > (SELECT AVG(total_laid_off) FROM layoffs_staging_2)
ORDER BY total_laid_off DESC;
```
![Querry-4](/assets/images/querry-10.PNG)

