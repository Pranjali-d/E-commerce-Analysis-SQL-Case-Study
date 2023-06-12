# Analyzing Traffic Sources

</br>

## Traffic Source Analysis
- Understanding where the customers are coming from and which channels are driving the highest quality traffic. 
- Typically, traffic sources are email, social media, search engine, and direct traffic. 
- Looking at conversion rate (CVR) which is the percentage of the traffic that converts into sales or revenue activity.


#### Why is it important to conduct Conversion Rate Analysis?
- Shift budget towards search engines, campaigns, or keywords driving the highest conversion rate.
- Compare user behaviour across traffic sources to customize messaging strategy.
- Identify opportunities to eliminate paid marketing channel or scale performing traffic.

*** 
</br>


### 📌 Q1: Finding Top Traffic Sources
<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171317382-d6cb4387-3a0b-4c72-a2fc-a0425fe359ca.png">

- **Stakeholder (ST)'s request:** Breakdown of sessions by UTM source, campaign and referring domain up to 12-04-2012
- **Results:** utm_source | utm_campaign | http_referer | sessions (count)
- **Steps:** Filter results up to sessions before '2012-04-12' and group results by `utm_source`, `utm_campaign` and `http_referer`

```sql
SELECT 
  utm_source, 
  utm_campaign, 
  http_referer,
  COUNT(website_session_id) AS total_sessions
FROM website_sessions
WHERE created_at < '2012-04-12' -- Take sessions up to 2012-04-12 only
GROUP BY utm_source, utm_campaign, http_referer -- Group results
ORDER BY total_sessions DESC; -- Sort total sessions by descending showing the highest to lowest session count
```

<img width="421" alt="Screenshot 2022-05-26 at 2 07 42 PM" src="https://user-images.githubusercontent.com/81607668/170427566-cc8b5f7c-72b2-4076-91b3-2725c5c67461.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318070-1151876c-31a6-40ea-ab20-4ac26a8d3623.png">

**Insights:** Drill into gsearch nonbrand campaign traffic to explore potential optimization opportunities.

***
</br>


### 📌 Q2: Traffic Conversion Rates
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318096-8facb274-2d69-468f-adc5-0fc6056a8bc0.png">

- **ST request:** Calculate CVR from session to order. If CVR is 4% >=, then increase bids to drive volume, otherwise reduce bids.
- **Results:** sessions (count) | orders (count) | conversion rate
- **Steps:** Filter sessions < 2012-04-12, `utm_source` = gsearch and `utm_campaign` = nonbrand

```sql
SELECT 
  COUNT(DISTINCT wb.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS session_to_order_cvr
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-04-14'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand';
```

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171318139-08f4d78b-a070-46f2-9b99-1c919f466ece.png">

**Insights:** Conversion rate is 2.92%, which is less than 4%, hence we're overspending on gsearch nonbrand campaign and have to reduce bids. To monitor impact of bid reduction for the campaign.

***
</br>


## Bid Optimization & Trend Analysis

Analysing for bid optimization is understanding the value of various segments of paid traffic to optimize marketing budget.
- Our job is to figure out the right amount of bid for various segments of traffic based on our potential revenue. 
- How do we do that?
  - Use conversion rate and revenue per click to figure out how much you should spend per click to acquire customers. 
  - Understand how website and products are performing for subsegments of traffic to optimise within channels. 
  - Analyse impact of bid changes have on ranking 

***
</br>

### 📌 Q3: Traffic Source Trending
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318231-7deec300-c768-4029-98b0-53acd7bb684f.png">

- **ST request:** After bidding down on Apr 15, 2021, what is the trend and impact on sessions for `gsearch` `nonbrand` campaign? Find weekly sessions before 2012-05-10. _I'm going a step forward and provide the conversion rate as well in anticipation that ST will ask for this information._
- **Result:** week_start | session (count) | orders (count) | conversion_rate
- **Steps:** Filter to < 2012-05-10, utm_source = gsearch, utm_campaign = nonbrand

```sql
SELECT
  MIN(DATE(wb.created_at)) AS week_start, -- 1st day of each week
	COUNT(DISTINCT wb.website_session_id) AS sessions, -- Unique count of sessions
	COUNT(DISTINCT o.order_id) AS orders, -- Unique count of successful orders resulting from the sessions
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS conversion_rate -- Conversion rate of orders from sessions
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-05-10'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY WEEK(wb.created_at);
```

<img width="275" alt="image" src="https://user-images.githubusercontent.com/81607668/139573298-51e1fb02-1a8c-4746-ba67-925a2405853a.png">

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318289-ab49b997-1c41-40e2-98f8-4d6ad0e6f239.png">

**Insights**: The sessions and conversion rate after 2021-04-15 has dropped - the campaign is highly sensitive to bid changes. Continue to monitor session volume. We want to make campaigns more efficient by maximising volume at the lowest possible bid.

***
</br>

### 📌 Q4: Traffic Source Bid Optimization
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171318335-32002c91-8990-42e1-8a93-b0196c5ceb70.png">

- **ST request:** What is the conversion rate from session to order by device type?
- **Result:** device_type | sessions (count) | orders (count) | conversion_rate

```sql
SELECT
  wb.device_type,
	COUNT(DISTINCT wb.website_session_id) AS sessions,
	COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT wb.website_session_id),2) AS conversion_rate
FROM website_sessions wb
LEFT JOIN orders o
	ON wb.website_session_id = o.website_session_id
WHERE wb.created_at < '2012-05-11'
	AND wb.utm_source = 'gsearch'
  AND wb.utm_campaign = 'nonbrand'
GROUP BY wb.device_type; -- Group by device type
```

<img width="260" alt="image" src="https://user-images.githubusercontent.com/81607668/139573682-d2ab9eb0-1cd7-457a-884f-bd58ccf1c738.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171318369-5fe5b121-c2ee-4a20-b148-87d7bc6a877f.png">

**Insights:** Desktop bids were driving nearly 4% of session to successful orders rate, so we should reduce mobile bids and transfer the paid traffic spent to desktop channel instead.

***
</br>

### 📌 Q5: Traffic Source Segment Trending
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171318395-a4a722b8-41f6-4a9d-a376-6eb9adde159b.png">

- **ST request:** After bidding up on desktop channel on 2012-05-19, what is the weekly session trend for both desktop and mobile?
- **Result:** week_start_date | desktop_sessions | mobile_sessions
- **Steps:** Filter to between 2012-04-15 to 2012-06-19, utm_source = gsearch, utm_campaign = nonbrand

```sql
SELECT
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) AS desktop_sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions
FROM website_sessions
WHERE created_at BETWEEN '2012-04-15' AND '2012-06-09'
	AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand'
GROUP BY WEEK(created_at);
```

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/139574114-532ab3a1-457b-4559-9130-6df7c237b932.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318424-4b33a3e3-8b43-4a34-a875-6778da3542f4.png">

**Insights:** After biding up desktop channel on 2012-05-19, there were obvious increase in desktop volume whereas mobile volume has also dropped considerably. ST made the right decision to focus on desktop and able to optimize spend efficiently.

***
</br>
