Netflix Movies and TV Shows Data Analysis using SQL
<img width="688" height="417" alt="image" src="https://github.com/user-attachments/assets/16134184-21f4-4b6e-a4e1-8479cc334c46" />
Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

Objectives

Examine the distribution of content types, including movies and TV shows.
Identify and compare the most common content ratings across movies and TV shows.
Analyze content trends based on release years, countries of origin, and duration.
Explore and categorize content using defined criteria and relevant keywords.

Dataset we used in this project : https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download

Schema
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

Business Problems and Solutions
1. Count the Number of Movies vs TV Shows
SELECT 
    type,
    COUNT(*) AS total_count
FROM netflix
GROUP BY type;


Objective: Analyze the overall distribution of content types available on Netflix.

2. Find the Most Common Rating for Movies and TV Shows
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;


Objective: Identify the most frequently assigned rating for each content type.

3. List All Movies Released in a Specific Year (e.g., 2020)
SELECT *
FROM netflix
WHERE release_year = 2020;


Objective: Retrieve all movies released in a given year to analyze annual content trends.

4. Find the Top 5 Countries with the Most Content on Netflix
SELECT *
FROM (
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY country
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;


Objective: Determine which countries contribute the most content to Netflix.

5. Identify the Longest Movie
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;


Objective: Find the movie with the maximum runtime.

6. Find Content Added in the Last 5 Years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';


Objective: Analyze recently added content to understand current catalog trends.

7. Find All Movies and TV Shows Directed by 'Rajiv Chilaka'
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';


Objective: List all content directed by a specific director.

8. List All TV Shows with More Than 5 Seasons
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;


Objective: Identify long-running TV shows with more than five seasons.

9. Count the Number of Content Items in Each Genre
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre;


Objective: Analyze content volume across different genres.

10. Find the Top 5 Years with the Highest Average Content Release in India
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 
        2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;


Objective: Identify years with the highest average contribution of Indian content.

11. List All Movies Classified as Documentaries
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries';


Objective: Retrieve all documentary movies available on Netflix.

12. Find All Content Without a Director
SELECT *
FROM netflix
WHERE director IS NULL;


Objective: Identify content entries missing director information.

13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;


Objective: Analyze recent appearances of a specific actor.

14. Find the Top 10 Actors with the Most Appearances in Indian Movies
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS appearance_count
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY appearance_count DESC
LIMIT 10;


Objective: Identify the most frequently appearing actors in Indian-produced content.

15. Categorize Content Based on 'Kill' and 'Violence' Keywords
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' 
              OR description ILIKE '%violence%' 
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;


Objective: Classify content based on sensitive keywords and analyze distribution.

Findings and Conclusion :
Content Diversity: Netflix hosts a wide variety of movies and TV shows across multiple genres and ratings.
Audience Targeting: Common rating patterns help identify the platform’s primary audience segments.
Geographical Trends: Country-wise analysis highlights regional content dominance, with India showing notable release patterns.
Content Nature: Keyword-based categorization provides insight into the thematic nature of available content.
