## Amazon_project_SQl
# Amazon Sales Analysis Project
![amazon_india_wide_image-3](https://github.com/user-attachments/assets/84478561-74e6-48ea-80e3-0c0b0042da38)

 Welcome to the Amazon Sales Analysis project! In this project, we delve into analyzing sales
 data from Amazon to extract insights and trends that can help optimize sales strategies,
 understand customer behavior, and improve business operations.

  ## Introduction
  
   This project focuses on analyzing a dataset containing Amazon sales records, including
   information such as sales dates, customer details, product categories, and revenue figures. By
   leveraging SQL queries and data analysis techniques, we aim to answer various questions and
   uncover valuable insights from the dataset.

  ## Dataset Overview
     The dataset used in this project consists of[insert number]  rows of data, representing Amazon
     sales transactions. Along with the sales data, the dataset includes information about customers,
     products, orders, and returns. Before analysis, the dataset underwent preprocessing to handle
     missing values and ensure data quality
     
 ## Analysis Questions Resolved
 During the analysis, the following key questions were addressed using SQL queries and data
 analysis techniques:

 ```SQL
1. Find out the top 5 customers who made the highest profits.
```SQL	
SELECT
       CUSTOMER_ID,
       SUM(SALE-PRICE_PER_UNIT) AS PROFIT
FROM ORDERS
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```		
 2. Find out the average quantity ordered per category.
```SQL
SELECT
       CATEGORY,
       AVG(QUANTITY) AS AVERAGE_QUANTITY
FROM ORDERS
GROUP BY 1 
```
3. Identify the top 5 products that have generated the highest revenue.
```SQL	 
SELECT
       PRODUCT_ID,
       SUM(SALE) AS TOTAL_REVENUE
 FROM ORDERS
 GROUP BY 1
 ORDER BY 2
 LIMIT 5 
```	 	 
4.Determine the top 5 products whose revenue has decreased compared to the previous year.
```SQL
WITH CURRENT_YEAR
AS	 
	(
		 SELECT
				PRODUCT_ID,
				SUM(SALE) AS CURRENT_YEAR_REVENUE,
				TO_CHAR(ORDER_DATE ,'YYYY') AS CURRENT_YEAR
		FROM ORDERS
		GROUP BY 1,3
		ORDER BY 2,3
		LIMIT 5 
	 ),
PREVIOUS_YEAR
	 AS
(

	 SELECT
	 		PRODUCT_ID,
	EXTRACT(YEAR FROM ORDER_DATE)-1 AS PREVIOUS_YEAR,
	 -- TO_CHAR(ORDER_DATE,'YYYY')- INTERVAL 1 AS PREVIOUS_YEAR,
	 		SUM(SALE)AS PREVIOUS_YEAR_REVENUE
	 		
 	FROM ORDERS
	GROUP BY 1,2
 	ORDER BY 2,3 DESC
 	LIMIT 5
)

SELECT 
		C.PRODUCT_ID,
		C.CURRENT_YEAR_REVENUE,
		C. CURRENT_YEAR,
	 	P.PRODUCT_ID,
		P.PREVIOUS_YEAR_REVENUE,
		P.PREVIOUS_YEAR,
	 	(C.CURRENT_YEAR_REVENUE-P.PREVIOUS_YEAR_REVENUE) AS REVENUE_DIFFERENCE
FROM CURRENT_YEAR AS C
JOIN PREVIOUS_YEAR AS P
ON C.PRODUCT_ID = P.PRODUCT_ID
WHERE 	(C.CURRENT_YEAR_REVENUE-P.PREVIOUS_YEAR_REVENUE)<0
-- GROUP BY REVENUE_DIFFERENCE 
LIMIT 5;		
```

5. Identify the highest profitable sub-category.
```SQL
SELECT
      SUB_CATEGORY,
      SUM(SALE-PRICE_PER_UNIT)AS TOTAL_PROFIT
FROM ORDERS
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```
6. Find out the states with the highest total orders.
```SQL
SELECT
       STATE,
       COUNT(ORDER_ID)AS TOTAL_ORDERS
FROM ORDERS
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```
7. Determine the month with the highest number of orders.
```SQL
SELECT	
     TO_CHAR(ORDER_DATE,'MONTH')AS MONTH,
     COUNT(ORDER_ID)AS NUM_OF_ORDERS
FROM ORDERS
GROUP BY 1
ORDER BY 2
LIMIT 1;
```

 8. Calculate the profit margin percentage for each sale (Profit divided by Sales).
```SQL
PROFIT = SALE -COGS* QUANTITY
	SELECT * FROM ORDERS

	SELECT * FROM PRODUCTS

SELECT
		O.SALE,
		 O.PRODUCT_ID,
	 	P.PRODUCT_NAME,
		SUM(O.SALE-(P.COGS * O.QUANTITY)) AS PROFIT,
	 	SUM(O.SALE-(P.COGS * O.QUANTITY))*100 AS PROFIT_PERCENTAGE
	 	
FROM ORDERS AS O 
JOIN PRODUCTS AS P
ON O.PRODUCT_ID = P.PRODUCT_ID
GROUP BY 1,2,3
ORDER BY 4
```
 9. Calculate the percentage contribution of each sub-category

```SQL
WITH total_sales AS (
    SELECT SUM(SALE) AS total_revenue
    FROM ORDERS
)
SELECT sub_category, 
       SUM(SALE) AS sub_category_revenue, 
       (SUM(SALE) / (SELECT total_revenue FROM total_sales)) * 100 AS percentage_contribution
FROM ORDERS
GROUP BY sub_category;
```
10.Identify top 2 category that has received maximum returns and their return %
```SQL
SELECT * FROM ORDERS;
SELECT * FROM RETURNS;

SELECT 
		CATEGORY,
		-- COUNT(ORDER_ID) AS TOTAL_ORDERS,
		COUNT(RETURN_ID) AS  TOTAL_RETURNS,
		(COUNT(RETURN_ID)/COUNT(ORDER_ID))*100 AS RETIRN_PERCENTAGE
FROM ORDERS AS O 
JOIN RETURNS AS R
ON O.ORDER_ID = R.ORDER_ID
GROUP BY 1
HAVING COUNT(RETURN_ID)
LIMIT 2 ;
```

## Conclusion
Through this project, we aim to provide valuable insights into Amazon sales trends, customer
 preferences, and other factors influencing e-commerce operations. By analyzing the dataset
 and addressing the key questions, we hope to assist stakeholders in making informed decisions
 and optimizing their sales strategies.
## Notice:
 All customer names and data used in this project are computer-generated using AI and random
 functions. They do not represent real data associated with Amazon or any other entity. This
 project is solely for learning and educational purposes, and any resemblance to actual persons,
 businesses, or events is purely coincidental.
