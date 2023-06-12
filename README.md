# 📊 Maven Fuzzy Factory (e-commerece store) Analysis 
![SQL](https://img.shields.io/badge/MySQL-4479A1.svg?style=for-the-badge&logo=MySQL&logoColor=white) 

</br>

# Introduction	

### 👩🏻‍💼 THE SITUATION 
You’ve just been hired as an eCommerce Database Analyst for Maven Fuzzy Factory, an online retailer which has just launched their first product.

</br>

### 📈 THE BRIEF
As a member of the startup team, you will work with the CEO, the Head of Marketing, and the Website Manager to help steer the business. 
You will analyze and optimize marketing channels, measure and test website conversion performance, and use data to understand the impact of new product launches.

</br>

### ✏️ THE OBJECTIVE
Use SQL to:
- Access and explore the Maven Fuzzy Factory database
- Analyze and optimize the business’ marketing channels, website, and product portfolio

	
***
</br>

# Entity Relationship Diagram
	
<kbd><img src="https://user-images.githubusercontent.com/81607668/139528817-09766746-e26b-4aa6-9465-bd5cc58a34cc.png" alt="Image" width="750" height="480"></kbd>
  
`orders` - Records consist of customers' orders with order id, time when the order is created, website session id, unique user id, product id, count of products purchased, price (revenue), and cost in USD. 

<kbd><img width="659" alt="image" src="https://user-images.githubusercontent.com/81607668/139528238-dade7402-2d4e-4a66-911e-ad1bd949f2ed.png"></kbd>

`order_items` - Records show various items ordered by customer with order item id, when the order is created, whether it is primary or non-primary item, product info, individual product price, and cost in USD. 

<kbd><img width="531" alt="image" src="https://user-images.githubusercontent.com/81607668/139528127-64ad1eff-abdb-4e3d-add5-47477ee0d84d.png"></kbd>

`order_item_refunds` - Refund information including when creation date and time and refund amount in USD.

<kbd><img width="527" alt="image" src="https://user-images.githubusercontent.com/81607668/139528283-926c7663-bdfb-4649-ab37-8c2ca814f47d.png"></kbd>

`products` - Product id, creation date of product in system, and product name.

<kbd><img width="330" alt="image" src="https://user-images.githubusercontent.com/81607668/139528155-b1c44957-d369-41af-a9d7-3da3928f0564.png"></kbd>

`website_sessions` - Table is showing where the traffic is coming from and which source is helping to generate the orders. Records consist of unique website session id, UTM (Urchin Tracking Module) fields. UTMs tracking parameters used by Google Analytics to track paid marketing activity. 

<kbd><img width="792" alt="image" src="https://user-images.githubusercontent.com/81607668/139528190-361da1c5-6512-4df7-860e-e0fdbd45f097.png"></kbd>

`website_pageviews`

<kbd><img width="427" alt="image" src="https://user-images.githubusercontent.com/81607668/139528173-eb792b62-c98a-47aa-bd80-bfa5c432063b.png"></kbd>

***
</br>





# Analysis 
- [Analyzing Traffic Sources](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/Analyzing%20Traffic%20Sources.md)
	- Top traffic sources
	- Conversion rates
	- Bid optimizations and Trend Analysis
- [Analyzing Website Performances](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/02_Analyzing%20Website%20Performance.md)
	- Analyzing top website pages and entry pages
	- Analyzing bounce rates and lading page tests
	- Building conversion funnels and testing conversion paths
- [Mid-Course Project](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/03_mid_course_project.md)
- [Analysis for Channel Portfolio Management]
 	- Analyzing channel portfolios
	- Comparing channel characteristics
	- Cross-channel bid optimization
	- Analyzing direct, brand-driven traffic
- [Analyzing Business Patterns and Seasonality]
	- Analyzing seasonality
	- Analyzing business patterns
- [Product Analysis]
	- Analyzing product sales and product launches
	- Analyzing product-level website pathing
	- Building product-level conversion funnels
	- Cross-sell analysis
	-Analyzing product refund rates
- [User Analysis]
	- Analyzing repeat visit and purchase behavior
- [Final Project]
 
***
</br>








## Analysis for Channel Portfolio Management

Analyzing a portfolio of marketing channels is about bidding efficiently and using data to maximise the effectiveness of your marketing budget.
- Understanding which marketing channels are driving the most sessions and orders through your website.
- Understanding differences in user characteristics and conversion performance across marketing channels.
- Optimising bids and allocating marketing spend across multi-channel portfolio to achieve marketing performance.

### Analyzing Channel Portfolios

### 📌 Q13: Analyzing Channel Portfolios 
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171320103-01635bd5-154f-4bda-b619-b75e79e5f5d6.png">

- **ST request:** Pull weekly sessions from 22 Aug to 29 Nov for gsearch and bsearch
- **Result:** weekly | gsearch_sessions | bsearch_sessions

```sql
SELECT
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_sessions,
  COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/ -- Added additional conversion rate
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS bsearch_rate
FROM website_sessions
WHERE utm_campaign = 'nonbrand'
  AND created_at BETWEEN '2012-08-22' AND '2012-11-29'
GROUP BY WEEK(created_at);
```

<img width="368" alt="image" src="https://user-images.githubusercontent.com/81607668/170911693-dae09a00-2c5f-4fe2-81bf-536f30eea6fa.png">

<img width="294" alt="image" src="https://user-images.githubusercontent.com/81607668/171320142-3ae5a7ba-c5cb-48f3-9fea-9472f69bdd3a.png">

**Insights:** Conversion rate is showing a consistent 3x impact of new bsearch campaign over original gsearch campaigns in the past 3 months. 

### 📌 Q14: Comparing Channel Characteristics

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171320158-e1cef2d3-93bc-4322-827b-1ae3ae6874cc.png">

- **ST request:** Pull mobile sessions for gsearch and bsearch non brand campaign from 22 Aug - 30 Nov
- **Result:** utm_source | sessions | mobile_sessions | mobile_perc

```sql
SELECT
  utm_source,
  COUNT(DISTINCT website_session_id) AS sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) AS mobile_sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END)/ 
    COUNT(DISTINCT website_session_id) AS mobile_perc
FROM website_sessions
WHERE created_at BETWEEN '2012-08-22' AND '2012-11-30'
  AND utm_campaign = 'nonbrand'
  AND utm_source IN ('gsearch', 'bsearch')
GROUP BY utm_source;
```

<img width="358" alt="image" src="https://user-images.githubusercontent.com/81607668/170913941-cd559385-f654-4c61-99c3-dee3232109bd.png">

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171320194-e45ee7fe-5555-4811-aa99-ba43fdc11b6d.png">

### 📌 Q15: Cross-Channel Bid Optimization
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171320250-76563d99-943e-414c-b2b2-e4902aeda20c.png">

- **ST request:** Pull gsearch and bsearch nonbrand conversion rates from session to orders and slice by device type from 22 Aug - 18 Sep
- **Result:** device_type | utm_source | sessions | orders | conversion_rate

```sql
SELECT 
  device_type,
  utm_source,
  COUNT(DISTINCT s.website_session_id) AS sessions,
  COUNT(DISTINCT o.order_id) AS orders,
  COUNT(DISTINCT o.order_id)/
    COUNT(DISTINCT s.website_session_id) AS conversion_rate
FROM website_sessions s
LEFT JOIN orders o
  ON s.website_session_id = o.website_session_id
WHERE s.utm_campaign = 'nonbrand'
  AND s.created_at BETWEEN '2012-08-22' AND '2012-09-18'
GROUP BY device_type, utm_source;
```

<img width="434" alt="image" src="https://user-images.githubusercontent.com/81607668/170936255-6103f002-22ca-4bdd-9215-81477cd8a824.png">

**Insights:** `gsearch` campaign has higher conversion rate in both desktop and mobile at 4.75% and 1.16% compared to 3.71% and 0.8% for `bsearch` campaign. Suggest to bid down on 'bsearch' campaigns.

### 📌 Q16: Channel Portfolio Trends
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320267-066e5b60-c81b-468e-b8fa-50d5f71c668d.png">

- **ST request:** Pull gsearch and bsearch nonbrand sessions by device type from 4 Nov - 22 Dec
- **Result:** week_start_date | device_type | utm_source | sessions | bsearch_comparison

```sql
SELECT 
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_desktop_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_desktop_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS b_perc_of_g_desktop,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS gsearch_mob_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END) AS bsearch_mob_sess,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'bsearch' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' AND utm_source = 'gsearch' THEN website_session_id ELSE NULL END) AS b_perc_of_g_mob
FROM website_sessions
WHERE created_at > '2012-11-04' AND created_at < '2012-12-22'
  AND utm_campaign = 'nonbrand'
GROUP BY WEEK(created_at);
```

<img width="767" alt="image" src="https://user-images.githubusercontent.com/81607668/170945303-9ca68616-8450-49c4-8253-f34ec3962fd6.png">

<img width="287" alt="image" src="https://user-images.githubusercontent.com/81607668/171320377-2472b08d-4f24-4cfb-8995-c5df48b3f7f4.png">

**Insights:**
- The desktop `bsearch` sessions was consistent at 4% of `gsearch` sessions, but dropped after reduced bids on 2 December. However, it could also be impacted by Black Friday, Cyber Monday and other seasonality factors. 
- As for mobile sessions, there was a sharp fall after bids were reduced, but was fluctuating throughout December, so it was hard to isolate whether it was due to reduced bids or other factors as well.

### Analying Direct Traffic

### 📌 Q17: Analyzing Free Channels
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320410-60dd7279-1b6a-44ee-83ff-da49fb1871ed.png">

- **ST request:** Pull organic search, direct type in and paid brand sessions by month. Present in % of paid nonbrand
- **Result:** yr | mo | nonbrand | brand | brand_pct_of_nonbrand | direct | direct_pct_of_nonbrand | organic | organic_pct_of_nonbrand

```sql
SELECT
  YEAR(created_at) AS yr,
  MONTH(created_at) AS mo,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS nonbrand,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id ELSE NULL END) AS brand,
  COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS brand_pct_of_nonbrand,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL and http_referer IS NULL THEN website_session_id ELSE NULL END) AS direct,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL and http_referer IS NULL THEN website_session_id ELSE NULL END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS direct_pct_of_nonbrand,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IN ('https://www.bsearch.com', 'https://www.gsearch.com') THEN website_session_id END) AS organic,
  COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IN ('https://www.bsearch.com', 'https://www.gsearch.com') THEN website_session_id END)/
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_session_id ELSE NULL END) AS organic_pct_of_nonbrand
FROM website_sessions
WHERE created_at < '2012-12-23'
GROUP BY YEAR(created_at), MONTH(created_at);
```

<img width="696" alt="image" src="https://user-images.githubusercontent.com/81607668/170958982-da5530ed-b4e4-4ca3-aeea-b7099f9e4853.png">

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171320439-477ac083-9bc0-496b-b69e-fdf046416d3c.png">

**Insights:**

***

## Analyzing Seasonality & Business Patterns

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

### 📌 Q19: Analyzing Business Patterns
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171320711-90fc2f5c-2ef6-434d-a8af-2d41a153351e.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320724-e1741577-6c30-45ec-bc88-bd7991af729e.png">

**Insights:**

***

## Product Analysis

### 📌 Q20: Product Level Sales Analysis
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320944-87fc4400-1153-4a8d-9493-4b692ad8073e.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171320976-23ec861c-37f5-4fe8-a675-94d316d5da01.png">

**Insights:**

### 📌 Q21: Product Launch Sales Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321008-b69e6b38-9cbf-42bd-aa42-01b88a635cb1.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321028-cc786a9c-91d5-4f21-b108-d81e45f4cbff.png">

**Insights:**

### Product Level Website Analysis

### 📌 Q22: Product Pathing Analysis
<img width="287" alt="image" src="https://user-images.githubusercontent.com/81607668/171321218-7c8db4f4-3384-48c5-b5a1-d2b86a1186a2.png">

- **ST request:**
- **Result:**

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171321239-e0b4aed0-d2a2-4418-b96c-a4d435ffcb0c.png">

**Insights:**

### 📌 Q23: Product Conversion Funnels
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321285-5df87937-7fd9-48fb-a103-3790de1d793a.png">

- **ST request:**
- **Result:**

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171321307-771b508e-4ac9-4c5c-afb8-2cfa240a88a1.png">

**Insights:**

### Cross-Selling Products

### 📌 Q24: Cross-Sell Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321353-c792df1b-b851-41ae-93df-1ff136874b05.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321373-9f314744-af27-4f91-9881-74cbf3d34951.png">

**Insights:**

### 📌 Q25: Portfolio Expansion Analysis
<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321393-c9de555f-e2ed-4302-af15-f0dd306e3878.png">

- **ST request:**
- **Result:**

<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171321438-1b4ac1c9-e81b-4cfb-b89e-7b935ad904f6.png">

**Insights:**

### Product Refund Analysis

### 📌 Q26: Product Refund Rates
<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171321511-85b37542-2653-41d8-9298-7db26e5ad7c1.png">

- **ST request:**
- **Result:**

<img width="289" alt="image" src="https://user-images.githubusercontent.com/81607668/171321546-834cd702-d32d-45c1-b49a-9a946321f52d.png">

**Insights:**

***

## User Analysis

### Analyse Repeat Behaviour

### 📌 Q27: Identifying Repeat Visitors
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171321676-ac59d791-ef96-45ac-be26-be66d5611139.png">

- **ST request:**
- **Result:**

<img width="292" alt="image" src="https://user-images.githubusercontent.com/81607668/171321700-27f961a7-6ad4-47a0-9f66-fe13bcd4aa79.png">

**Insights:**

### 📌 Q28: Analyzing Repeat Behaviour
<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171321756-020a5bf9-04b8-4299-a61c-8163b2ff966d.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321784-c057525c-034d-459e-be13-dcbd0be45559.png">

**Insights:**

### 📌 Q29: New Vs. Repeat Channel Patterns
<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321840-d2724912-2e76-45a7-bda6-9cfb1c240d59.png">

- **ST request:**
- **Result:**

<img width="290" alt="image" src="https://user-images.githubusercontent.com/81607668/171321872-41c1cee2-2a0a-4b2e-992f-7ca762d22c73.png">

**Insights:**

### 📌 Q30: New Vs. Repeat Performance
<img width="293" alt="image" src="https://user-images.githubusercontent.com/81607668/171321900-c182791a-6d0f-4bd7-ae65-da6e7df8b42e.png">

- **ST request:**
- **Result:**

<img width="288" alt="image" src="https://user-images.githubusercontent.com/81607668/171322014-a4376283-9cbe-44ec-91f0-775b983b8090.png">

**Insights:**

***

## Final Project

<img width="291" alt="image" src="https://user-images.githubusercontent.com/81607668/171322246-c494af57-1fd1-4009-a8fc-9dec714e6f9c.png">

**👩🏻‍💼 THE SITUATION** 

Cindy is close to securing Maven Fuzzy Factory’s next round of funding, and she needs your help to tell a compelling story to investors. You’ll need to pull the relevant data, and help your CEO craft a story about a data-driven company that has been producing rapid growth.

**📈 THE OBJECTIVE**

Extract and analyze **traffic and website performance data** to craft a **growth story** that your CEO can sell. Dive in to the **marketing channel activities and the website improvements** that have contributed to your success to date, and use the opportunity to flex your analytical skills for the investors while you’re at it.

As an Analyst, the first part of your job is extracting and analyzing the data. The next (equally important) part is **communicating the story effectively to your stakeholders.**

### 📌 Q1: First, I’d like to show our volume growth. Can you pull overall session and order volume, trended by quarter for the life of the business? Since the most recent quarter is incomplete, you can decide how to handle it.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q2: Next, let’s showcase all of our efficiency improvements. I would love to show quarterly figures since we launched, for session-to-order conversion rate, revenue per order, and revenue per session.
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q3: I’d like to show how we’ve grown specific channels. Could you pull a quarterly view of orders from Gsearch nonbrand, Bsearch nonbrand, brand search overall, organic search, and direct type-in?
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q4: Next, let’s show the overall session-to-order conversion rate trends for those same channels, by quarter. Please also make a note of any periods where we made major improvements or optimizations.
- **ST request:**
- **Result:**

**Insights:**

### 📌 Q5: We’ve come a long way since the days of selling a single product. Let’s pull monthly trending for revenue and margin by product, along with total sales and revenue. Note anything you notice about seasonality.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q6: Let’s dive deeper into the impact of introducing new products. Please pull monthly sessions to the /products page, and show how the % of those sessions clicking through another page has changed over time, along with a view of how conversion from /products to placing an order has improved.

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q7: We made our 4th product available as a primary product on December 05, 2014 (it was previously only a cross-sell item). Could you please pull sales data since then, and show how well each product cross-sells from one another?

- **ST request:**
- **Result:**

**Insights:**

### 📌 Q8: In addition to telling investors about what we’ve already achieved, let’s show them that we still have plenty of gas in the tank. Based on all the analysis you’ve done, could you share some recommendations and opportunities for us going forward? No right or wrong answer here – I’d just like to hear your perspective!

- **ST request:**
- **Result:**

**Insights:**

***
