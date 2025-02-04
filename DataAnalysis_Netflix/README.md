# 🎬 Netflix Data Exploration with PostgreSQL

## 📌 Overview
This project involves analyzing a **Netflix dataset** using **PostgreSQL Workbench**. The dataset, originally in `.csv` format, was imported into PostgreSQL after creating a structured database and table. The goal of this project was to explore the data and solve **15 key business problems** using **SQL queries, Common Table Expressions (CTEs), functions, and window functions**.

---

## 📂 Project Workflow
### 🏗️ 1. **Database & Table Creation**
- A new **PostgreSQL database** named `netflix_db` was created. ✅
- A table named `netflix` was structured with appropriate **columns and data types** to match the dataset.

### 📥 2. **Data Import**
- The `.csv` dataset was imported into `netflix` 📊
- Ensured proper **data integrity** and handled missing values where applicable.

### 🔍 3. **Data Analysis**
- Used SQL queries to extract insights and solve **15 business-related problems**. 📊

---

## 🔑 Key Business Questions & Solutions

### 📌 **Schema**
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id VARCHAR(6),
	show_type VARCHAR(10),
	title VARCHAR(150),
	director VARCHAR(208),
	casts VARCHAR(1000),
	country VARCHAR(150),
	date_added VARCHAR(50),
	release_year INT,
	rating VARCHAR(10),
	duration VARCHAR(15),
	listed_in VARCHAR(100),
	description VARCHAR(250)
);
```

### **1. Count the number of Movies and Tv Shows**
**Query:**
```sql
SELECT show_type, COUNT(show_type) AS number_of_items
FROM netflix
GROUP BY show_type
ORDER BY show_type;
```

### **2. Find the most common rating for movies and tv shows**
**Query:**
```sql
WITH t1 AS
	(SELECT show_type, 
			rating, 
			COUNT(rating) AS rating_count,
			RANK() OVER(PARTITION BY show_type ORDER BY COUNT(rating) DESC) AS ranking
	FROM netflix
	GROUP BY show_type, rating
)
SELECT show_type, rating
FROM t1
WHERE ranking = 1;
```
### **3. List all movies released in a specific year (eg. 2020)**
**Query:**
```sql
SELECT show_type, title
FROM netflix
WHERE show_type = 'Movie' AND release_year = 2020;
```
### **4. Find the top 5 countries with the most content on netflix**
**Query:**
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
	COUNT(show_id) AS total_content
FROM netflix
GROUP BY country
ORDER BY total_content desc
LIMIT 5;
```
### **5. Identify the longest movie**
**Query:**
```sql
WITH t1 AS(
	SELECT title, 
		CAST(REGEXP_REPLACE(duration, '[^0-9]', '','g')AS INTEGER) AS duration_mins
	FROM netflix
	WHERE show_type = 'Movie'
)
SELECT *
FROM t1
WHERE duration_mins = (SELECT MAX(duration_mins) FROM t1);
```
### **6. Find content added in the last five years**
**Query:**
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 Years';
```
### **7. Find all the movies or TV Shows by director 'Rajiv Chilaka'**
**Query:**
```sql
SELECT *
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';
```
### **8. List all Tv Shows with more than 5 seasons**
**Query:**
```sql
WITH t1 AS(
	SELECT *, 
		CAST(REGEXP_REPLACE(duration, '[^0-9]', '','g')AS INTEGER) AS no_of_seasons
	FROM netflix
	WHERE show_type = 'TV Show'
)
SELECT *
FROM t1
where no_of_seasons > 5;
```
### **9. Count the number of content items in each genre**
**Query:**
```sql
SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, 
	COUNT(show_id) AS content_count
FROM netflix
GROUP BY genre
ORDER BY 1;
```
### **10. Find each year and the percentage of content released in India on netflix also Return top 5 year with highest percent content release**
**Query:**
```sql
SELECT 
	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD YYYY')) AS years,
	COUNT(show_id) as total_content,
	ROUND(COUNT(show_id) :: numeric / (SELECT COUNT(show_id) FROM netflix WHERE country = 'India') * 100, 2):: numeric AS percent_count
FROM netflix
WHERE country = 'India'
GROUP BY years
ORDER BY percent_count DESC
LIMIT 5;
```
### **11. List all movies that are documentaries**
**Query:**
```sql
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```
### **12. Select content from netflix where there is no director involved**
**Query:**
```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```
### **13. Find how many movies the actor Salman Khan appeared in the last 10 years**
**Query:**
```sql
SELECT COUNT(*) as movies_appeared
FROM netflix
WHERE show_type = 'Movie' AND
	casts ILIKE ('%Salman Khan%') AND
	TO_DATE(date_added, 'Month DD YYYY') >= CURRENT_DATE - INTERVAL '10 Years';
```
### **14. Find the top 10 actors who have appeared in the highest number of movies produced in India**
**Query:**
```sql
SELECT
	UNNEST(STRING_TO_ARRAY(casts, ',')) AS actors,
	COUNT(show_id) as no_of_movies
FROM netflix
WHERE country ILIKE ('%India') AND show_type ILIKE ('Movie') AND casts IS NOT NULL 
GROUP BY actors
ORDER BY no_of_movies DESC
LIMIT 10;
```
### **15. Categorize the contents based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many times items fall into each category.**
**Query:**
```sql
WITH t1 AS (
	SELECT
		*,
		CASE
			WHEN description ILIKE '%kill%' OR 
			description ILIKE '%violence%' OR 
			description ILIKE '%violent%' OR 
			description ILIKE '%killed%' 
			then 'Bad'
			ELSE 'Good'
		END AS category
	FROM netflix
)
SELECT category, COUNT(category) AS category_count
FROM t1
GROUP BY category;
```
