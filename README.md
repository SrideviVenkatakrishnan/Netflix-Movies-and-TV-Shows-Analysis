# Netflix-Movies-and-TV-Shows-Analysis
Using PostgreSQL

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset
The data for this project is sourced from the Kaggle dataset:

- Dataset Link: [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions
### 1. Count the Number of Movies vs TV Shows
```sql
SELECT 
    type,
    COUNT(*) AS movie_count
FROM netflix
GROUP BY 1;
```
### 2. Find the most common rating for movies and TV shows
```sql
SELECT
	type, rating
FROM (
	SELECT
		type, rating, count(*),
		rank() over (partition by type order by count(*) desc) as rnk
	FROM netflix
	GROUP BY 1, 2
	)
WHERE rnk = 1;
```
### 3. List all movies released in a specific year
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
### 4. Find the top 5 countries with the most content on Netflix
```sql
SELECT
	TRIM(UNNEST(STRING_TO_ARRAY(country, ','))), 
	COUNT(show_id)
FROM netflix
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```
### 5. Identify the longest movie
```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
	AND duration IS NIT NULL
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
```
### 6. Find content added in the last five years
```sql
SELECT *
FROM netflix
WHERE
	date_added::DATE >= CURRENT_DATE - INTERVAL '5 years';
```
### 7. Find all the movies by director 'Rajiv Chilaka'
```sql
SELECT *
FROM (
    SELECT 
        *,
        TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```
### 8. List all TV shows with more than 5 seasons
```sql
SELECT *
FROM netflix
WHERE type = 'TV Show' 
	AND TRIM(SUBSTRING(duration, 1, 2))::NUMERIC > 5;
```
### 9. Count the number of content items in each genre
```sql
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC;
```
### 10. For each year find the average numbers of content released in the United States on Netflix.
### Return top 5 year with highest avg content release.
```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'United States')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'United States'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```
### 11. List all movies that are documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```
### 12. Find all content without a director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
### 13. Find how many movies actor 'Morgan Freeman' appeared in last 10 years
```sql
SELECT * 
FROM netflix
WHERE casts ILIKE '%Morgan Freeman%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
  AND type = 'Movie';
```
### 14. Find out the top 10 actors who have appeared in the highest number of movies produced in India
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
	AND type = 'Movie'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```
### 15. Categorize the content based on the presence of keywords 'kill' and 'violence' in the description field.
### Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items
### fall into each category
```sql
WITH cte as (
	SELECT *,
		CASE
			WHEN description ilike '%kill%'
				 OR description ilike '%violence%' THEN 'Bad Content'
			ELSE 'Good Content'
		END AS category
	FROM netflix
)
SELECT category, count(*)
FROM cte
GROUP BY category;
```
