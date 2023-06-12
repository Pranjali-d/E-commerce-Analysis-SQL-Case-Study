# Analyzing Website Performances

Website content analysis is about understanding which pages are seen most by the users and to identify where to focus on improving your business.
- Identify the most common landing page and the first thing a user sees.
- For most viewed pages and common landing pages, understand how those pages perform for business objectives.
- Does the pages tell a story or resonate with business values?

In this section, we will learn about
- Analyzing top website pages and entry/landing pages
- Analyzing bounce rates and landing pages test
- Building conversion funnels and testing conversion paths

***
</br>

### 📌 Q6: Identifying Top Website Pages
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171318612-5344a53c-052a-4647-8e11-0c6d8e9cf215.png">

- **ST request:** Identify most viewed website pages ranked by session volume
- **Result:** pageview_url | sessions (count)
- **Steps:** Find page view count by page view url, filter date < 2012-06-09 and sort session count descencing

```sql
SELECT 
  pageview_url, 
  COUNT(DISTINCT website_pageview_id) AS page_views
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY page_views DESC;
```

<img width="216" alt="image" src="https://user-images.githubusercontent.com/81607668/139636941-dfca8add-4466-454a-8d76-3c5e9139bded.png">

<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171318642-72591bb0-7067-44c3-9149-5802e80298e5.png">

- **Insights:** Most viewed pages with highest traffic are **homepage, products, and original Mr Fuzzy**. Is traffic for top landing pages the same?

***
</br>

### 📌 Q7: Identifying Top Entry Pages
<img width="295" alt="image" src="https://user-images.githubusercontent.com/81607668/171318667-f31c2781-c7fd-4cbe-821a-8a1a085cfa6c.png">

Entry/landing page is the page where customer lands on website for the first time.
- **ST request:** Pull a list of top entry pages
- **Result:** landing_page_url | landing_page_count
- **Steps:** Filter date to < 2012-06-12

```sql
WITH landing_page_cte AS (
SELECT
  website_session_id,
  MIN(website_pageview_id) AS landing_page_count -- Find the first page landed for each session
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id
)

SELECT 
  wp.pageview_url AS landing_page_url,
  COUNT(DISTINCT lp.website_session_id) AS count
FROM landing_page_cte lp
LEFT JOIN website_pageviews wp
  ON lp.landing_page = wp.website_pageview_id
GROUP BY wp.pageview_url
ORDER BY landing_page_url DESC;
```

<img width="153" alt="image" src="https://user-images.githubusercontent.com/81607668/139637250-f60c1773-59e3-4708-9eae-3c9cd0b795fc.png">

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171318697-24420909-8d62-4433-b350-9dd56cc698e8.png">

- **Insights:** Home page is the top landing page. What other metrics can we use to analyse landing page performance? How do we know the metric can appropriately judge whether a page is performing well or not?
- **Possible metrics:** Repeat sessions? Which day and time most viewed? By source, campaign, device type?

***
</br>

## Landing Page Performance and Testing

### 📌 Q8: Calculating Bounce Rates
<img width="296" alt="image" src="https://user-images.githubusercontent.com/81607668/171318738-a557166e-97bb-432b-be56-6b005e85e4b5.png">

Now that we have the sessions for the landing pages, let's find out their bounce rate.

We breakdown the steps to get a clear picture of what we're trying to find.
- **Result:** landing page | sessions (count) | bounced sessions (count) | bounced rate
- **Step 1:** Find the first `website_pageview_id` or landing page for associated session
- **Step 2:** Count page views for each session to identify bounces
- **Step 3:** Summarize total sessions and bounced sessions and calculate bounce rate

```sql
-- Step 1: Find first website_pageview_id/landing page for associated session and filter to date < 2012-06-14 and '\home'
WITH landing_page_cte AS (
SELECT 
  p.website_session_id,
  MIN(p.website_pageview_id) AS first_landing_page_id,
  p.pageview_url AS landing_page -- page view of first landing page
FROM website_pageviews p
INNER JOIN website_sessions s
  ON p.website_session_id = s.website_session_id
  AND s.created_at < '2012-06-14'
WHERE p.pageview_url = '/home' 
GROUP BY p.website_session_id
),
```
<img width="326" alt="image" src="https://user-images.githubusercontent.com/81607668/171318902-6972b40c-91bc-489e-adcd-b7f1876f44d3.png">

```sql
- Step 2: Count page views for each session to identify bounces
bounced_views_cte AS (
SELECT 
  lp.website_session_id,
  lp.landing_page, 
  COUNT(p.website_pageview_id) AS bounced_views
FROM landing_page_cte lp
LEFT JOIN website_pageviews p
  ON lp.website_session_id = p.website_session_id
GROUP BY 
  lp.website_session_id,
  lp.landing_page
HAVING COUNT(p.website_pageview_id) = 1
)
```
<img width="309" alt="image" src="https://user-images.githubusercontent.com/81607668/171319068-2597bb86-eabc-4fd9-8a2d-b7ea97a2d925.png">

```sql
- Step 3: Summarize total sessions and bounced sessions
SELECT 
  COUNT(DISTINCT lp.website_session_id) AS total_sessions, -- number of sessions by landing page
  COUNT(DISTINCT b.website_session_id) AS bounced_sessions, -- number of bounced sessions by landing page
  ROUND(100 * COUNT(DISTINCT b.website_session_id)/
    COUNT(DISTINCT lp.website_session_id),2) AS bounce_rate
FROM landing_page_cte lp -- use left join to preserve all sessions with 1 home page view
LEFT JOIN bounced_views_cte b
  ON lp.website_session_id = b.website_session_id
GROUP BY lp.landing_page;
```

<img width="268" alt="image" src="https://user-images.githubusercontent.com/81607668/139804071-9f7b4dc3-4bed-46e0-bc59-89b90b3b9747.png">

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171318771-cca1bc9a-b5ed-4847-bd56-6ed91d2a392c.png">

**Insights:** 60% bounce rate is pretty high especially for paid search.

***
</br>

### 📌 Q9: Analyzing Landing Page Tests
<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171319348-c663000e-69b4-4c89-8193-a48c9853675c.png">

ST is running a A/B test on `\lander-1` and `\home` for `gsearch nonbrand` campaign and would like to find out the bounce rates for both pages.
- **Criteria:** Limit time period to when `\lander-1` started receiving traffic and limit results to < 2012-07-28 to ensure fair comparison.
- **Result:** landing_page | total sessions | bounced sessions | bounce rate
- **Step 1:** Find when `/lander-1` was created and first displayed to user on the website. Use either date or pageview id to limit the results.
- **Step 2:** Find first landing page and filter to test time period (after '2012-06-19') and as prescribed by ST (before '2012-07-28') for gsearch and nonbrand campaign.
- **Step 3:** Count page views for each session to identify bounces
- **Step 4:** Summarize total sessions and bounced sessions and calculate bounce rate

```sql
-- Step 1: Find when `/lander-1` was created and first displayed to user on the website
SELECT 
  MIN(created_at) AS lander1_created_at,
  MIN(website_pageview_id) AS lander1_website_pageview_id
FROM website_pageviews
WHERE pageview_url = '/lander-1';

-- Step 2: Find first landing page and filter to test time period '2012-06-19' to '2012-07-28' for gsearch and nonbrand campaign
WITH landing_page_cte AS (
SELECT 
  p.website_session_id,
  MIN(p.website_pageview_id) AS landing_page_id,
  p.pageview_url AS landing_page -- page view of first landing page
FROM website_pageviews p
INNER JOIN website_sessions s
  ON p.website_session_id = s.website_session_id
  AND s.created_at BETWEEN '2012-06-19' AND '2012-07-28' -- A/B test time period as prescribed
  AND utm_source = 'gsearch'
  AND utm_campaign = 'nonbrand'
  AND p.pageview_url IN ('/home','/lander-1') -- A/B test on both pages
GROUP BY p.website_session_id, p.pageview_url
),
```
<img width="301" alt="image" src="https://user-images.githubusercontent.com/81607668/171319187-d0635e13-12e8-40f2-a130-b0e18bbf7e75.png">

```sql
-- Step 3: Count page views for each session to identify bounces
bounced_views_cte AS (
SELECT 
  lp.website_session_id,
  COUNT(p.website_pageview_id) AS bounced_views
FROM landing_page_cte lp
LEFT JOIN website_pageviews p
  ON lp.website_session_id = p.website_session_id -- join where session id with first landing page
GROUP BY lp.website_session_id
HAVING COUNT(p.website_pageview_id) = 1 -- Filter for page views = 1 view = bounced view
)
```
<img width="233" alt="image" src="https://user-images.githubusercontent.com/81607668/171319265-62a7e901-3958-49cf-abe9-ccfef22b5f73.png">

```sql
-- Step 4: Summarize total sessions and bounced sessions and calculate bounce rate
SELECT 
  landing_page,
  COUNT(DISTINCT lp.website_session_id) AS total_sessions, -- number of sessions by landing page
  COUNT(DISTINCT b.website_session_id) AS bounced_sessions, -- number of bounced sessions by landing page
  ROUND(100 * COUNT(DISTINCT b.website_session_id)/
    COUNT(DISTINCT lp.website_session_id),2) AS bounce_rate
FROM landing_page_cte lp -- use left join to preserve all sessions with 1 page view
LEFT JOIN bounced_views_cte b
  ON lp.website_session_id = b.website_session_id
GROUP BY lp.landing_page;
```

<img width="343" alt="image" src="https://user-images.githubusercontent.com/81607668/139808742-1f307189-6633-4765-baf2-57969405f8cd.png">

<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171319373-59adc93b-7be6-4141-849e-a43f04ff5cdb.png">

**Insights:** Looks like the newly created `/lander-1`'s traffic has improved and bounce rate has reduced too, meaning fewer customers has bounced on the page.  

**Next steps:** Ensure that all new campaigns are directed to the new lander-1 page and monitor the bounce rates.

***
</br>


### 📌 Q10: Landing Page Trend Analysis
<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319399-477106ef-b077-4ed7-920c-190ff5299d81.png">

- **ST request:** Pull paid gsearch nonbrand campaign traffic on `/home` and `/lander-1` pages, trended weekly since 2012-06-01 and the bounce rates.
- **Criteria:** Email received on 31 Aug 2021, so limit results between 2012-06-01 to 2012-08-31
- **Result:** landing page | week start dates | sessions (count) | bounced sessions (count) | bounce rate | 

- **Step 1:** Find first website_pageview_id for first landing page id
- **Step 2:** Identify landing page of each session
- **Step 3:** Count page views for each session to identify bounces
- **Step 4:** Summarize sessions, bounced sessions and bounce rate by week 

```sql
-- Step 1: Find first website_pageview_id/landing page and no of page views
WITH landing_pages_cte AS (
SELECT 
  s.website_session_id,
  MIN(p.website_pageview_id) AS first_pageview_id, -- first page view id
  COUNT(p.website_pageview_id) AS pageview_count -- no of total view counts per session
FROM website_sessions s 
INNER JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at BETWEEN '2012-06-01' AND '2012-08-31'
  AND s.utm_source = 'gsearch'
  AND s.utm_campaign = 'nonbrand'
GROUP BY s.website_session_id
),
```
<img width="297" alt="image" src="https://user-images.githubusercontent.com/81607668/171319449-a8a5da76-9885-4af9-ab9d-a5e556b0a1bd.png">

```sql
),
summary_cte AS (
SELECT
  lp.website_session_id,
  lp.first_pageview_id,
  lp.pageview_count,
  p.pageview_url AS landing_page, -- add new field
  p.created_at -- add new field
FROM landing_pages_cte lp
INNER JOIN website_pageviews p
  ON lp.first_pageview_id = p.website_pageview_id
)
```
<img width="525" alt="image" src="https://user-images.githubusercontent.com/81607668/171319543-6cbc7cb1-0d75-4b59-96fa-b0d208245706.png">

```sql
SELECT
  YEARWEEK(created_at) AS year_week,
  MIN(DATE(created_at)) AS week_start, -- 1st day of associated week
  COUNT(DISTINCT website_session_id) AS total_sessions,
  COUNT(DISTINCT CASE WHEN pageview_count = 1 THEN website_session_id END) AS bounced_sessions,
  ROUND(100 * COUNT(DISTINCT CASE WHEN pageview_count = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS bounce_rate,
  COUNT(DISTINCT CASE WHEN landing_page = '/home' THEN website_session_id ELSE NULL END) AS home_sessions,
  COUNT(DISTINCT CASE WHEN landing_page = '/lander-1' THEN website_session_id ELSE NULL END) AS lander_sessions
FROM summary_cte s
GROUP BY WEEK(created_at);
```

<img width="558" alt="image" src="https://user-images.githubusercontent.com/81607668/170437950-75753ca5-4d34-42de-9bd5-491cd58588fd.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319595-c6cf09eb-0eea-41e0-887b-0c9fb055bf9e.png">

**Insights:** Before 2012-06-17, all traffic were routed to home, then after 2012-08-05 all traffic is routed to lander-1. Bounce rate dropped from 60%+ to nearing 50% so there is improvement. Changes made to `/lander-1` page is working well.

***
</br>

## Analyzing and Testing Conversion Funnels
Conversion funnel analysis is about understanding and optimizing each step of user's experience on their journey towards purchasing our products.

**Homepage > Product Page > Add to Cart > Sale**

- Identify most common paths customers take before purchasing our products.
- Identify how many of the users continue to the next step in the conversion funnel and how many users abandon at each step.
- Optimize critical pain points where users are abandoning so that you can convert more users and sell more products.


</br>

### 📌 Q11: Build Conversion Funnels for `gsearch nonbrand` traffic from `/lander-1` to `/thank you` page
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319628-a14f9841-56f0-496a-b1d9-389a077c3988.png">

- **Criteria:** Limit results from 2012-08-05 to 2012-09-05
- **Result:** sessions | lander1 click % | product click % | mrfuzzy click % | cart click % | shipping click % | billing click % | thank you click %
- **Step 1:** Select all pageviews fr relevant session
- **Step 2:** Identify each relevant pageview as specific funnel step
- **Step 3:** Create session-level conversion funnel view
- **Step 4:** Aggregate data to assess funnel performance

```sql
-- Step 1: Select all pageviews fr relevant session
WITH pageview_cte AS (
SELECT 
  s.website_session_id,
  p.pageview_url,
  CASE WHEN p.pageview_url = '/products' THEN 1 ELSE 0 END AS to_products,
  CASE WHEN p.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS to_mrfuzzy,
  CASE WHEN p.pageview_url = '/cart' THEN 1 ELSE 0 END AS to_cart,
  CASE WHEN p.pageview_url = '/shipping' THEN 1 ELSE 0 END AS to_shipping,
  CASE WHEN p.pageview_url = '/billing' THEN 1 ELSE 0 END AS to_billing,
  CASE WHEN p.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS to_thankyou
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE s.created_at BETWEEN '2012-08-05' AND '2012-09-05'
  AND s.utm_campaign = 'nonbrand'
  AND s.utm_source = 'gsearch'
),
```
<img width="647" alt="image" src="https://user-images.githubusercontent.com/81607668/171319726-9c0cfae3-d9a1-48c4-87c8-78b46d84b0e6.png">

```sql
-- Step 2: Identify each relevant pageview as specific funnel step
-- Step 3: Create session-level conversion funnel view
summary_cte AS (
SELECT
  website_session_id, 
  MAX(to_products) AS product_views,
  MAX(to_mrfuzzy) AS mrfuzzy_views,
  MAX(to_cart) AS cart_views,
  MAX(to_shipping) AS shipping_views,
  MAX(to_billing) AS billing_views,
  MAX(to_thankyou) AS thankyou_views
FROM pageview_cte
GROUP BY website_session_id
)
```
<img width="683" alt="image" src="https://user-images.githubusercontent.com/81607668/171319827-fb6c151d-d7e1-40d9-93c0-ac58292bf205.png">

```sql
-- Step 4: Aggregate data to assess funnel performance
SELECT
  COUNT(DISTINCT website_session_id) AS sessions, -- total sessions
  ROUND(100 * COUNT(CASE WHEN product_views = 1 THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT website_session_id),2) AS products_click_rate, --   product views / total sessions
  ROUND(100 * COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN product_views = 1 THEN website_session_id ELSE NULL END),2) AS mrfuzzy_click_rate, -- mrfuzzy views / product views
  ROUND(100 * COUNT(CASE WHEN cart_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END),2) AS cart_click_rate, -- cart views / mrfuzzy views
  ROUND(100 * COUNT(CASE WHEN shipping_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN mrfuzzy_views = 1 THEN website_session_id ELSE NULL END),2) AS shipping_click_rate, -- shipping views / cart views
  ROUND(100 * COUNT(CASE WHEN billing_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN shipping_views = 1 THEN website_session_id ELSE NULL END),2) AS billing_click_rate, -- billing views / shipping views
  ROUND(100 * COUNT(CASE WHEN thankyou_views = 1 THEN website_session_id ELSE NULL END)/ 
    COUNT(CASE WHEN billing_views = 1 THEN website_session_id ELSE NULL END),2) AS cart_click_rate -- thankyou views / billing views
FROM summary_cte;
```

<img width="677" alt="image" src="https://user-images.githubusercontent.com/81607668/170442720-694c553e-e962-492f-a3ba-e981929d8c4e.png">

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319870-fbeb1dc4-aefe-4d88-8181-8d107e4111ec.png">

**Insights: **
- To focus on /lander-1, mrfuzzy and billing pages with lowest clickthrough rate - why are users dropping off on these pages?
- More information on billing page will make customers more comfortable to insert their credit card information.

***
</br>

### 📌 Q12: Analyze Conversion Funnel Tests for `/billing` vs. new `/billing-2` pages
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171319887-cf73ec4b-1b9e-401d-8e31-c533d9bb9dd8.png">

- **ST request:** ST developed a new `/billing-2` page and wants to test the traffic and billing to order conversion rate of both pages.
- **Result:** billing_version | sessions | orders | billing_to_order_rate

```sql
-- Step 1: Find when /billing-2 was first active
SELECT 
  MIN(website_pageview_id)
FROM website_pageviews
WHERE pageview_url = '/billing-2';
-- /billing-2 is first active on website pageview id = 53550

-- Step 2: Select all pageviews for relevant session
WITH billing_cte AS (
SELECT 
  s.website_session_id,
  p.pageview_url
FROM website_sessions s
LEFT JOIN website_pageviews p
  ON s.website_session_id = p.website_session_id
WHERE p.website_pageview_id >= 53550 -- first page view when /billing-2 is active
  AND s.created_at < '2012-11-10'
  AND p.pageview_url IN ('/billing', '/billing-2')
)
-- Step 3: Aggregate and summarise the conversion rate
SELECT
  b.pageview_url,
  COUNT(DISTINCT b.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  ROUND(100 * COUNT(DISTINCT o.order_id)/COUNT(DISTINCT b.website_session_id),2) AS session_to_orders_rate
FROM billing_cte b
LEFT JOIN orders o
  ON b.website_session_id = o.website_session_id
GROUP BY b.pageview_url;
```

<img width="317" alt="image" src="https://user-images.githubusercontent.com/81607668/140033607-c8f07fcf-a56a-460f-ad8d-4e8a0da2bccb.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171319913-0004fe00-957f-4453-98c5-361f81d8cd30.png">

**Insights:** `/billing-2` page has session to order converstion rate at 62%; much better than billing page at 46%. To request engineering team to roll out new  `/billing-2` page to customers immediately.

**Next step:** Monitor overall sales performance.

***
</br>
