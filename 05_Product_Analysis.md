# Analyzing Seasonality & Business Patterns
</br>


### 📌 Q18: Analyzing Seasonality
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320599-d3a19ced-b3ed-4e3e-b904-ba6f8d69798d.png">

- **ST request:** Pull out sessions and orders by year, monthly and weekly for 2012
- **Result:** Table 1 - yr | mo| sessions | orders | order_rate; Table 2 - week_start | sessions | orders | order_rate

```sql
SELECT 
  YEAR(s.created_at) AS yr,
  MONTH(s.created_at) AS mo,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/COUNT(DISTINCT s.website_session_id) AS order_rate -- added a column to observe trend
FROM website_sessions s
LEFT JOIN orders o 
  ON s.website_session_id = o.website_session_id
WHERE YEAR(s.created_at) = 2012 -- Limit to year 2012 only
GROUP BY YEAR(s.created_at), MONTH(s.created_at);
```

<img width="263" alt="image" src="https://user-images.githubusercontent.com/81607668/171373410-7b635c8c-42f8-42b8-be8a-43eaae69aca0.png">

```sql
SELECT 
  MIN(WEEK(s.created_at)) AS week_start,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/COUNT(DISTINCT s.website_session_id) AS order_rate -- added a column to observe trend
FROM website_sessions s
LEFT JOIN orders o 
  ON s.website_session_id = o.website_session_id
WHERE YEAR(s.created_at) = 2012
GROUP BY WEEK(s.created_at);
```

<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171373703-d6db366c-b506-423e-8540-d896cc612335.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171320624-af4fbe70-93a2-40c9-a6b7-cd548f0e0151.png">

**Insights:**

***
</br>

### 📌 Q19: Analyzing Business Patterns
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320711-90fc2f5c-2ef6-434d-a8af-2d41a153351e.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320724-e1741577-6c30-45ec-bc88-bd7991af729e.png">

**Insights:**

***
