# Layoffs Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](tools)
- [Data Cleaning](#data-cleaning)
- [Questions](#questions)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Limitations](#limitations)

### Project Overview

This project analyzes the total laid off across the world by different company’s and industries from year 2020 to 2023, and also to show trends and patterns by each year.

### Data Source

Data: The primary dataset used for this analysis is the [layoffs](https://github.com/onatolumayowa/Layoffs-Analysis-using-SQL/blob/main/layoffs.csv) containing detailed information about the world layoffs.

### Tools

- MySQL - Data Cleaning And Data Analysis
    - [Download here](https://dev.mysql.com/downloads/workbench/)

### Data Cleaning

In the initial data prepation phase, I performed the following task:

- Data loading

- Checking for duplicates/Handling duplicates
```sql
SELECT *
FROM layoffs
WHERE company = 'Oda';
```

```sql
WITH duplicate_cte AS
(
SELECT *, 
	ROW_NUMBER() OVER(PARTITION BY company, industry, total_laid_off,
  percentage_laid_off, date,
  stage, country, funds_raised_millions) AS row_num
FROM layoffs
)
SELECT *
FROM duplicate_cte 
WHERE row_num > 1;
```

- Removing duplicates
```sql
CREATE TABLE `layoffs_stagging` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num`	INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

```sql
INSERT INTO layoffs_stagging
SELECT *, 
	ROW_NUMBER() OVER(PARTITION BY company, industry, total_laid_off, percentage_laid_off, date,
    stage, country, funds_raised_millions) AS row_num
FROM layoffs;
```

```sql
DELETE 
FROM layoffs_stagging
WHERE row_num > 1;
```

- Standardize the data

```sql
UPDATE layoffs_stagging
SET company = TRIM(company);
```

```sql
UPDATE layoffs_stagging
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

```sql
UPDATE layoffs_stagging
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

```sql
UPDATE layoffs_stagging
SET date = STR_TO_DATE(date, '%m/%d/%Y');
```

```sql
ALTER TABLE layoffs_stagging
MODIFY COLUMN date DATE;
```

- NULL values or blank values

```sql
UPDATE layoffs_stagging
SET industry = NULL
WHERE industry = '';
```

```sql
UPDATE layoffs_stagging t1
JOIN layoffs_stagging t2
	ON 	t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

- Remove any unwanted columns/rows

```sql
DELETE 
FROM layoffs_stagging
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

```sql
ALTER TABLE layoffs_stagging
DROP COLUMN row_num;
```

### Questions

- Identify the company with the highest total laid off
- Identify the year with the most total laid off
- Get the rolling total of laid off by each day
- Identify the company with the highest laid off in each year
- Show the top 5 company’s in each year with the highest total laid off

### Data Analysis

- Identify the company with the highest total laid off
```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_stagging
GROUP BY 1
ORDER BY 2 DESC;
```

- Identify the year with the most total laid off
```sql
SELECT YEAR(date), SUM(total_laid_off)
FROM layoffs_stagging
GROUP BY 1
ORDER BY 2 DESC;
```

- Get the rolling total of laid off by each day
```sql
WITH Rolling_Total AS
(
SELECT SUBSTRING(date, 1,7) AS month, SUM(total_laid_off) AS total_off
FROM layoffs_stagging
WHERE SUBSTRING(date, 1,7) IS NOT NULL
GROUP BY 1
ORDER BY 1 ASC
)
SELECT month, total_off, SUM(total_off) OVER(ORDER BY month) AS rolling_total
FROM Rolling_Total;
```

- Identify the company with the highest laid off in each year
```sql
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_stagging
GROUP BY 1, 2
)
SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
ORDER BY ranking;
```

- Show the top 5 company’s in each year with the highest total laid off
```sql
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_stagging5
GROUP BY 1, 2
), Company_Year_Rank AS
(
SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE ranking <= 5;
```

### Results

The Analysis results are summarized as follows:
1. After the analysis I observed that the company with the highest sum of total laid off is Amazon and this might be due to the over-population of the employees.
2. According to the dataset, 2022 had the highest number of laid off.
3. I observed that at the end of 2023, the highest total laid off should be more that that of year 2022.
4. After the analysis I observed that the country with the highest sum of total laid off is the United States and this might also be due to over-population
5. After the analysis I observed that laid off occurred mostly at the beginning of the year.


### Limitations

I had to remove rows of NULL values for both total_laid_off and percentage_laid_off because they would have affected the accuracy of my conclusion from the analysis. There are still few NULL values in some rows and i don't know if i should delete them or not beacuse i certainly don't know the total employees in each company.













   
   
