# DataLemur SQL Exercises

Solutions to DataLemur SQL exercises. 

### LinkedIn

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. You want to find candidates who are proficient in Python, Tableau, and PostgreSQL.

Write a query to list the candidates who possess all of the required skills for the job. Sort the output by candidate ID in ascending order.

```sql
SELECT candidate_id
FROM candidates
WHERE skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY candidate_id
HAVING COUNT(skill) = 3
ORDER BY candidate_id ASC;
```

### Facebook

Assume you are given the tables below about Facebook pages and page likes. Write a query to return the page IDs of all the Facebook pages that don't have any likes. The output should be in ascending order.

```sql
SELECT pages.page_id
FROM pages LEFT JOIN page_likes AS likes
  ON pages.page_id = likes.page_id
WHERE likes.page_id IS NULL
ORDER BY page_id;
```
### Tesla

Tesla is investigating bottlenecks in their production, and they need your help to extract the relevant data. Write a query that determines which parts have begun the assembly process but are not yet finished.

```sql
SELECT part 
FROM parts_assembly
WHERE finish_date IS NULL
GROUP BY part;
```

### New York Times

Assume that you are given the table below containing information on viewership by device type (where the three types are laptop, tablet, and phone). Define “mobile” as the sum of tablet and phone viewership numbers. Write a query to compare the viewership on laptops versus mobile devices.

Output the total viewership for laptop and mobile devices in the format of "laptop_views" and "mobile_views".

```sql
SELECT 
  SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END) AS laptop_views,
  SUM(CASE WHEN device_type = 'tablet' OR device_type = 'phone' THEN 1 ELSE 0 END) AS mobile_views
FROM
  viewership;
```

### Robinhood

You are given the tables below containing information on Robinhood trades and users. Write a query to list the top three cities that have the most completed trade orders in descending order.

```sql
SELECT users.city, COUNT(trades.order_id) AS total_orders
FROM trades JOIN users ON trades.user_id = users.user_id
WHERE trades.status = 'Completed'
GROUP BY users.city
ORDER BY total_orders DESC
LIMIT 3;
```

### LinkedIn

Assume you are given the table below that shows job postings for all companies on the LinkedIn platform. Write a query to get the number of companies that have posted duplicate job listings.

```sql
WITH jobs_grouped AS (
SELECT company_id, title, description, COUNT(job_id) AS job_count
FROM job_listings
GROUP BY company_id, title, description
) 

SELECT COUNT(DISTINCT company_id) AS companies_duplicate_jobs
FROM jobs_grouped
WHERE job_count > 1;
```
### Facebook

Given a table of Facebook posts, for each user who posted at least twice in 2021, write a query to find the number of days between each user’s first post of the year and last post of the year in the year 2021. Output the user and number of the days between each user's first and last post.

```sql
SELECT user_id, MAX(post_date::DATE) - MIN(post_date::DATE) AS days_between_posts
FROM posts
WHERE DATE_PART('year', post_date::DATE) = 2021
GROUP BY user_id
HAVING COUNT(post_id) >= 2;
```
### Microsoft

Write a query to find the top 2 power users who sent the most messages on Microsoft Teams in August 2022. Display the IDs of these 2 users along with the total number of messages they sent. Output the results in descending count of the messages.

Assumption:
No two users has sent the same number of messages in August 2022.

```sql
SELECT sender_id, COUNT(message_id) AS message_count
FROM messages
WHERE EXTRACT(MONTH FROM sent_date) = '8'
  AND EXTRACT(YEAR FROM sent_date) = '2022'
GROUP BY sender_id
ORDER BY message_count DESC
LIMIT 2;
```
