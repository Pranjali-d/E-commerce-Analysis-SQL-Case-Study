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
- [Analyzing Traffic Sources](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/01_Analyzing_Traffic_Sources.md)
	- Top traffic sources
	- Conversion rates
	- Bid optimizations and Trend Analysis
- [Analyzing Website Performances](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/02_Analyzing_Website_Performance.md)
	- Analyzing top website pages and entry pages
	- Analyzing bounce rates and lading page tests
	- Building conversion funnels and testing conversion paths
- [Mid-Course Project](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/03_mid_course_project.md)
- [Analysis for Channel Portfolio Management](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/04_Channel_portfolio_management.md)
 	- Analyzing channel portfolios
	- Comparing channel characteristics
	- Cross-channel bid optimization
	- Analyzing direct, brand-driven traffic
- [Analyzing Business Patterns and Seasonality](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/05_Buisness_pattern_and_Seasonality.md)
	- Analyzing seasonality
	- Analyzing business patterns
- [Product Analysis](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/06_Product_Analysis.md)
	- Analyzing product sales and product launches
	- Analyzing product-level website pathing
	- Building product-level conversion funnels
	- Cross-sell analysis
	-Analyzing product refund rates
- [User Analysis](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/07_User_Analysis.md)
	- Analyzing repeat visit and purchase behavior
- [Final Project](https://github.com/Pranjali-d/E-commerce-Analysis-SQL-Case-Study/blob/main/08_Final_project.md)
 
***
</br>

# Refrence
https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/















