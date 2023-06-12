# Mid-Course Project

</br>


### 👩🏻‍💼 THE SITUATION

Maven Fuzzy Factory has been live for ~8 months, and your CEO is due to present company performance metrics to the board next week. You’ll be the one tasked with preparing relevant metrics to show the company’s promising growth.

</br>

### ✏️ THE OBJECTIVE

Use SQL to:
- Extract and analyze website traffic and performance data from the Maven Fuzzy Factory database to **quantify the company’s growth**, and to tell the story of **how you have been able to generate that growth**.
- As an Analyst, the first part of your job is extracting and analyzing the data, and the next part of your job is effectively communicating the story to your stakeholders.

***
</br>

### 📌 Q1: Gsearch seems to be the biggest driver of our business. Could you pull monthly trends for gsearch sessions and orders so that we can showcase the growth there?

- **Table:** year month | sessions | orders | session to order rate

```sql
SELECT
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/ 
    COUNT(DISTINCT s.website_session_id),2) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="271" alt="image" src="https://user-images.githubusercontent.com/81607668/140636799-322905ad-5e44-492f-a5ad-009d23253bac.png">

**Insights:** Steady growth of gsearch session to order rate from 3.27% in Mar 2012 to 4.23% to Nov 2012. 

***
</br>

### 📌 Q2: Next, it would be great to see a similar monthly trend for Gsearch, but this time splitting out nonbrand and brand campaigns separately. I am wondering if brand is picking up at all. If so, this is a good story to tell.

- **Table:** year_month | nonbrand_sessions | nonbrand_orders | nonbrand_conversion_r | brand_sessions | brand_orders | brand_conversion_r

```sql
SELECT 
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN s.website_session_id ELSE NULL END) AS nonbrand_sessions,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END) AS nonbrand_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.utm_campaign = 'nonbrand' THEN s.website_session_id ELSE NULL END),2) AS nonbrand_cvr,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN s.website_session_id ELSE NULL END) AS brand_sessions,
  COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN o.order_id ELSE NULL END) AS brand_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.utm_campaign = 'brand' THEN s.website_session_id ELSE NULL END),2) AS brand_cvr
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign IN ('nonbrand', 'brand')
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="603" alt="image" src="https://user-images.githubusercontent.com/81607668/140636838-1a81092c-a7ac-4eba-abbc-aa46493d0f3c.png">

**Insights:** Nonbrand session to order rate is steadily growing from 3.28% to 4.19%. Brand conversion rate is slightly inconsistent with highest 9.84% in Apr 2012 and recently averaging at 4-5% in late 2012. Could be due to different brands being promoted in different month or season. It would help to have further breakdown of brands in the campaign.

***
</br>

### 📌 Q3: While we’re on Gsearch, could you dive into nonbrand, and pull monthly sessions and orders split by device type? I want to flex our analytical muscles a little and show the board we really know our traffic sources.

- **Table:** year_month | mobile_sessions | mobile_orders | mobile_cvr | desktop_sessions | desktop_orders | desktop_cvr

```sql
SELECT
  EXTRACT(YEAR_MONTH FROM s.created_at) AS yr_mth,
  COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN s.website_session_id ELSE NULL END) AS mobile_sessions,
  COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN o.order_id ELSE NULL END) AS mobile_orders, 
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.device_type = 'mobile' THEN s.website_session_id ELSE NULL END),2) AS mobile_cvr,
  COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN s.website_session_id ELSE NULL END) AS desktop_sessions,
  COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN o.order_id ELSE NULL END) AS desktop_orders,
  ROUND(100 * COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.device_type = 'desktop' THEN s.website_session_id ELSE NULL END),2) AS desktop_cvr
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
GROUP BY EXTRACT(YEAR_MONTH FROM s.created_at);
```

<img width="538" alt="image" src="https://user-images.githubusercontent.com/81607668/140636873-4db9e55e-3522-4d1e-a867-d6080c622b0b.png">

Insights: Bulk of session to orders are contributed by users on desktop devices with good growth from 4.45% in Mar 2012 to 5.04% in Nov 2021. Investigate why mobile rate is low - maybe not user-friendly or pages are not displayed properly on mobile phone browser. Consider moving bids to desktop to optimize traffic and marketing spend.

***
</br>

### 📌 Q4: I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?

- Table: yearmonth | sessions | gsearch_session | gsearch_rate | bsearch_session | bsearch_rate

```sql
SELECT
  DISTINCT utm_source,
  utm_campaign,
  http_referer
FROM website_sessions
WHERE created_at < '2012-11-27';
```
<img width="338" alt="image" src="https://user-images.githubusercontent.com/81607668/171320004-a26391a5-bc7a-417c-a983-c28a551a8608.png">

- If source, campaign and http referral is NULL, then it is direct traffic - users type in the website link in the browser's search bar.
- If source and campaign is NULL, but there is http referral, then it is organic search - coming from search engine and not tagged with paid parameters.

```sql 
SELECT
  EXTRACT(YEAR_MONTH FROM created_at) AS yr_mth,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_paid_session,
  COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_paid_session,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_session_id ELSE NULL END) AS organic_search_session,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_session_id ELSE NULL END) AS direct_search_session
FROM website_sessions
WHERE created_at < '2012-11-27'
GROUP BY EXTRACT(YEAR_MONTH FROM created_at);
```

<img width="652" alt="image" src="https://user-images.githubusercontent.com/81607668/140637462-2180a38d-c133-4b34-839c-1c6248f2514c.png">

Insights: Gsearch traffic started out high in Mar 2012 with 98%, then slowly dropped to 70%. Bsearch traffic picked up in Aug 2012 and grow to 22% in Nov 20122. There are 10-15% null traffic. Identify what it means by null traffic.

***
</br>

### 📌 Q5: I’d like to tell the story of our website performance improvements over the course of the first 8 months. Could you pull session to order conversion rates, by month?

- ST request: Calculate conversion rate of orders based on sessions
- Result: yearmonth | sessions | orders | conversion_rate

```sql
SELECT
  YEAR(s.created_at) AS yr,
  MONTH(s.created_at) AS mth,
  COUNT(DISTINCT CASE WHEN s.website_session_id IS NOT NULL THEN s.website_session_id ELSE NULL END) AS sessions,
  COUNT(DISTINCT CASE WHEN o.order_id IS NOT NULL THEN o.order_id ELSE NULL END) AS orders, 
  ROUND(100 * COUNT(DISTINCT CASE WHEN o.order_id IS NOT NULL THEN o.order_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN s.website_session_id IS NOT NULL THEN s.website_session_id ELSE NULL END),2) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.created_at < '2012-11-27'
GROUP BY YEAR(s.created_at), MONTH(s.created_at);
```

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/170808375-03f0a66c-4f2f-480b-886c-590e1ecd1625.png">

Insights: Conversion rate increased from 3.23% in March 2012 to 4.45% in November. Rates start to hit 4% in September.

***
</br>

### 📌 Q6: For the gsearch lander test, please estimate the revenue that test earned us (Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)

- Revenue from Jun 19 – Jul 28 vs same period

```sql
-- Find the first lander-1 website_pageview_id
SELECT
  MIN(website_pageview_id)
FROM website_pageviews
WHERE pageview_url = '/lander-1';
```

The first pageview url website_pageview_id = 23504

```sql
SELECT
  p.pageview_url AS landing_page,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT s.website_session_id),2) AS conversion_rate
FROM website_sessions s
INNER JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE p.website_pageview_id >= 23504
  AND s.created_at < '2012-07-28'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND p.pageview_url IN ('/home', '/lander-1')
GROUP BY p.pageview_url;
``` 
<img width="297" alt="image" src="https://user-images.githubusercontent.com/81607668/170808139-e5c195bc-b9e6-4c7c-aa7f-ceb11a71a269.png">

Homepage's conversion rate is 3.11% and the new page lander-1's conversion rate is 4.14%. Incremental difference in website performance is 1.03% using lander-1.

```sql
SELECT
  MAX(s.website_session_id)
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at < '2012-11-27'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND p.pageview_url = '/home';
-- the last website session with gsearch nonbrand paid campaign = 17145

SELECT
  COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE created_at < '2012-11-27'
  AND website_session_id > 17145 -- last home session
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand';
```

<img width="145" alt="image" src="https://user-images.githubusercontent.com/81607668/170808159-a39d180a-824f-47d4-bc53-f78e61386ee5.png">

**Insights:** 
- Improved total sessions using `\lander-1` = 21,729
- 21,729 x 1.03% (incremental % of order) = estimated at least 223 incremental orders since 29 Jul using `\lander-1` page
- 223/4 months = 55 additional orders per month!
- Increased performance of website and quantified the performance of the additional website sessions.
***
</br>


### 📌 Q7: For the landing page test you analyzed previously, it would be great to show a full conversion funnel from each of the two pages to orders. You can use the same time period you analyzed last time (Jun 19 – Jul 28).

```sql
CREATE TEMPORARY TABLE flagged_sessions_summary
SELECT
  website_session_id,
  MAX(homepage) AS saw_homepage,
  MAX(custom_lander) AS saw_custom_lander,
  MAX(products_page) AS product_made_it,
  MAX(mrfuzzy_page) AS mrfuzzy_page_made_it,  
  MAX(cart_page) AS cart_page_made_it,  
  MAX(shipping_page) AS shipping_page_made_it,
  MAX(billing_page) AS billing_page_made_it,  
  MAX(thankyou_page) AS thankyou_page_made_it 
FROM (
SELECT
  s.website_session_id,
  CASE WHEN p.pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
  CASE WHEN p.pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
  CASE WHEN p.pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
  CASE WHEN p.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
  CASE WHEN p.pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
  CASE WHEN p.pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
  CASE WHEN p.pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
  CASE WHEN p.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
  AND s.created_at BETWEEN '2012-06-19' AND '2012-07-28'
ORDER BY s.website_session_id, p.created_at) AS flagged_sessions
GROUP BY website_session_id;

SELECT
  CASE WHEN saw_homepage = 1 THEN 'saw_homepage'
    WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
    ELSE 'check logic' END AS segment,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
  COUNT(DISTINCT CASE WHEN mrfuzzy_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
  COUNT(DISTINCT CASE WHEN cart_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
  COUNT(DISTINCT CASE WHEN shipping_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
  COUNT(DISTINCT CASE WHEN billing_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
  COUNT(DISTINCT CASE WHEN thankyou_page_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM flagged_sessions_summary
GROUP BY segment;

-- Categorise website sessions under `segment` by 'saw_homepage' or 'saw_custom_lander'
-- Convert aggregated website sessions to percentage of click rate by dividing by total sessions
SELECT
  CASE WHEN saw_homepage = 1 THEN 'saw_homepage'
    WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
    ELSE 'check logic' END AS segment,
  COUNT(DISTINCT website_session_id) AS sessions,
  ROUND(100*COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS product_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN mrfuzzy_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS mrfuzzy_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN cart_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS cart_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN shipping_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS shipping_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN billing_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS billing_click_rt,
  ROUND(100*COUNT(DISTINCT CASE WHEN thankyou_page_made_it = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS thankyou_click_rt
FROM flagged_sessions_summary
GROUP BY segment;
```

<img width="693" alt="image" src="https://user-images.githubusercontent.com/81607668/170445488-afb17bea-b93b-4290-aa96-29f73eb99bac.png">

**Insights:** Custom lander page has overall better click through rate as compared to the original homepage. 

***
</br>

### 📌 Q8: I’d love for you to quantify the impact of our billing test, as well. Please analyze the lift generated from the test (Sep 10 – Nov 10), in terms of revenue per billing page session, and then pull the number of billing page sessions for the past month to understand monthly impact.

- ST request: Calculate the sessions for `/billing` and `/billing-2` and revenue per billing page from Sep 10 – Nov 10
- Result: billing_version | sessions (count) | revenue per billing session

```sql
-- Pull out relevant data ie. website session id, page url, order id and prices associated with the billing pages
WITH billing_revenue AS (
SELECT 
  p.website_session_id, 
  p.pageview_url AS billing_version, -- Page url associated with the website session id
  o.order_id, -- 
  o.price_usd -- Billing associated with each order id. It represents the revenue
FROM website_pageviews p
LEFT JOIN orders o
  ON p.website_session_id = o.website_session_id
WHERE p.created_at BETWEEN '2012-09-10' AND '2012-11-10'
  AND p.pageview_url IN ('/billing', '/billing-2'))
```

<img width="329" alt="image" src="https://user-images.githubusercontent.com/81607668/170806763-43e92bbb-88ae-42b9-bdc4-d5423305810a.png">

```sql
SELECT 
  billing_version,
  COUNT(DISTINCT website_session_id) AS sessions, -- 
  SUM(price_usd)/COUNT(DISTINCT website_session_id) AS revenue_per_billing_page
FROM billing_revenue
GROUP BY billing_version;
```
<img width="327" alt="image" src="https://user-images.githubusercontent.com/81607668/170806793-d4491997-3519-4697-891a-002cd9331d1d.png">

```sql
SELECT 
  COUNT(website_session_id) AS sessions
FROM website_pageviews
WHERE pageview_url IN ('/billing', '/billing-2')
  AND created_at BETWEEN '2012-10-27' AND '2012-11-27';
```

<img width="85" alt="image" src="https://user-images.githubusercontent.com/81607668/170807652-e6c50539-ded2-4da3-96bd-97b3e5a36304.png">

**Insights:**
- $23.04 for old '\billing' page and $31.31 for new '\billing-2' page. Lift of $8.27 per billing page view; increased by 35%.
- Over the past month, there are 1,021 sessions and with the increase of $8.27 average revenue per session, we are looking at a positive impact of $8,443.67 increase in revenue.

***
