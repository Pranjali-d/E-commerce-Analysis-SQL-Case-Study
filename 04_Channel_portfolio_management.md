# Analysis for Channel Portfolio Management
</br>


Analyzing a portfolio of marketing channels is about bidding efficiently and using data to maximise the effectiveness of your marketing budget.
- Understanding which marketing channels are driving the most sessions and orders through your website.
- Understanding differences in user characteristics and conversion performance across marketing channels.
- Optimising bids and allocating marketing spend across multi-channel portfolio to achieve marketing performance.

***
</br>


## Analyzing Channel Portfolios
</br>

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

***
</br>

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

***
</br>

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

***
</br>

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

***
</br>

## Analying Direct Traffic

</br>

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
</br>
