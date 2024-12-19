## Top 5 industries with the most layoffs ##
```sql
SELECT industry, sum(total_laid_off) as top_layoffs
FROM layoffs_staging_2
WHERE industry != "other"
GROUP BY industry
ORDER BY top_layoffs desc
LIMIT 5;
```

## Top 5 companies with the most layoff per year ##

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

## Most layoff by country outside of the United States ##

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







