# Netflix Movies and TV Shows Data Analysis using SQL

![Logo](https://github.com/shanthan98/Netflix_Analysis/blob/main/assets/images/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using PostgreSQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Experience with PostgreSQL

As a data analyst with extensive experience using SQL for various databases, this project marked my first time using PostgreSQL. Transitioning to PostgreSQL provided an opportunity to explore some of its unique features and capabilities, particularly in terms of advanced querying techniques. The PostgreSQL environment (PgAdmin) was intuitive and allowed for efficient handling of complex queries with its support for features such as window functions, subqueries, and Common Table Expressions (CTEs).

This project specifically focused on using PostgreSQL's robust functionality for working with text and dates, as well as advanced functions that made querying data easier and more efficient. Below are some of the concepts and functions I employed during this project:

### Concepts Used

- **Window Functions**: These allow you to perform calculations across a set of table rows that are related to the current row, enabling powerful analytical queries.
- **Subqueries**: Used to nest queries within queries, allowing for more modular and readable SQL code.
- **CTEs (Common Table Expressions)**: Provided a way to structure complex queries in a more readable and maintainable manner by temporarily defining result sets.

### Functions Used

1. **`EXTRACT()`**: This function extracts parts of a date, such as the year, month, or day, from a timestamp or date field.
   - **Example**: `EXTRACT(YEAR FROM date_added)` would return the year from a given date.
   
2. **`TO_DATE()`**: Converts a string to a date using a specified format.
   - **Example**: `TO_DATE('2021-12-01', 'YYYY-MM-DD')` converts the string to a date format.

3. **`ILIKE`**: Performs a case-insensitive search using pattern matching.
   - **Example**: `SELECT * FROM netflix WHERE title ILIKE '%action%'` would return titles that include 'action' in any case.

4. **`STRING_TO_ARRAY()`**: Converts a delimited string into an array.
   - **Example**: `STRING_TO_ARRAY('apple,banana,cherry', ',')` converts the string into an array of `['apple', 'banana', 'cherry']`.

5. **`UNNEST()`**: Expands an array into a set of rows.
   - **Example**: `UNNEST(STRING_TO_ARRAY('apple,banana,cherry', ','))` would return each element as a separate row.

6. **`SPLIT_PART()`**: Splits a string by a delimiter and returns the nth part of the string.
   - **Example**: `SPLIT_PART('apple-banana-cherry', '-', 2)` would return `'banana'`.

### Platform/Software:
- **PgAdmin**: Used for managing and querying PostgreSQL databases.
- **Language**: PostgreSQL

## Task

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

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
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
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
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

### Differences/Advantages/Observations of PostgreSQL Over SQL:

1. **Advanced Data Types**: PostgreSQL supports more advanced data types such as arrays, hstore (key-value pairs), and JSON, making it highly versatile for storing structured data.
  
2. **Window Functions**: PostgreSQL has powerful support for window functions, which provide advanced analytics capabilities that can be difficult to implement in traditional SQL databases.

3. **Case-Insensitive Searches (`ILIKE`)**: PostgreSQL provides `ILIKE` for case-insensitive searches, which simplifies querying when case does not matter, while in MySQL, you would need to use additional workarounds like `LOWER()`.

4. **Text Processing**: PostgreSQL excels in text processing with functions like `STRING_TO_ARRAY` and `SPLIT_PART`, which can make dealing with text-heavy data easier compared to other databases.

5. **Strong Compliance to Standards**: PostgreSQL adheres closely to the SQL standard, making it more feature-rich and flexible than many other relational database systems.

6. **CTEs and Recursion**: PostgreSQL’s support for recursive CTEs makes it easier to write hierarchical queries and perform operations that involve traversing tree-like structures.
