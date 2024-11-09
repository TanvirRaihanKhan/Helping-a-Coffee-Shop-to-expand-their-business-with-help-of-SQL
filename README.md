# Helping SRI Coffee to choose their physical outlet locations
## Objective
**The goal of this project is to analyze the sales data of SRI Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening their physical shops based on consumer demand and sales performance.**

## Recommendations
After analyzing the data, the recommended top three cities for new store openings & reasons behind them are:
  
**City 1: Pune**  

1)Highest total revenue generator.<br/>   
2)Low average rent per customer.<br/>  
3)Average sales per customer is very high.<br/>  

**City 2: Delhi**  

1)Highest estimated coffee consumers.<br/>   
2)2nd highest unique customers.<br/>  
3)Average rent per customer is 330 (which is under 500).<br/>  

**City 3: Jaipur** 

1)Highest unique customers.<br/>   
2)Lowest average rent per customer.<br/>    

![Company Logo](https://github.com/TanvirRaihanKhan/Helping-a-Coffee-Shop-to-expand-their-business-with-help-of-SQL/blob/main/cofee_store.jpg)

## Key questions to be answered before picking a city 
1. **Coffee Consumers Count**  
   How many people in each city are estimated to consume coffee, given that 25% of the population does?
```sql
 SELECT city_name,
        ROUND((population*0.25)/1000000,2) AS cofee_consumers_in_millions
 FROM city
 ORDER BY 2 DESC;
```
2. **Total Revenue from Coffee Sales**  
   What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
 ```sql
SELECT ci.city_name, 
       SUM(s.total) AS total_revenue
FROM sales AS s
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON ci.city_id=c.city_id
WHERE 
	EXTRACT(YEAR FROM s.sale_date) =2023
	AND
	EXTRACT(QUARTER FROM s.sale_date) =4
GROUP BY ci.city_name
ORDER BY 2 DESC
```
3. **Sales Count for Each Product**  
   How many units of each coffee product have been sold?
```sql
SELECT p.product_name,
       COUNT(s.sale_id) AS total_orders
FROM products AS p
LEFT JOIN sales AS s
ON s.product_id=p.product_id
GROUP BY 1
ORDER BY 2 DESC
```
4. **Average per customer sales per City**  
   What is the average sales amount per customer in each city?
```sql
SELECT ci.city_name, 
       SUM(s.total) AS total_sales,
       COUNT(DISTINCT s.customer_id) AS total_customer,
       ROUND(SUM(s.total)::numeric/COUNT(DISTINCT s.customer_id)::numeric,2) AS avg_sales_per_customer
FROM sales AS s
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON ci.city_id=c.city_id
GROUP BY ci.city_name
ORDER BY 2 DESC
```
5. **City Population and Coffee Consumers (25%)**  
   Provide a list of cities along with their populations and estimated coffee consumers.
```sql
SELECT ci.city_name,
       ROUND((ci.population*0.25)/1000000,2) AS coffee_consumers,
       COUNT(DISTINCT c.customer_id) AS unique_customers
 FROM city AS ci
 JOIN customers AS c
 ON ci.city_id=c.city_id
 GROUP BY 1,2
 ORDER BY 2 DESC
```
This can be done with CTEs as well-

```sql
WITH city_table AS
(SELECT city_name,
        ROUND((population*0.25)/1000000,2) AS coffee_consumers
 FROM city),
 
customers_table AS 
(SELECT ci.city_name,
        COUNT(DISTINCT c.customer_id) AS unique_customers
 FROM sales AS s
 JOIN customers AS c
 ON s.customer_id=c.customer_id
 JOIN city AS ci
 ON ci.city_id=c.city_id
 GROUP BY 1)

SELECT customers_table.city_name,
       city_table.coffee_consumers AS coffee_consumers_in_million,
       customers_table.unique_customers
FROM city_table
JOIN customers_table
ON city_table.city_name=customers_table.city_name
ORDER BY 2 DESC
```
6. **Top Selling Products by City**  
   What are the top 3 selling products in each city based on sales volume?
```sql
SELECT city_name,
       product_name,total_orders
FROM
(
SELECT ci.city_name,
       p.product_name,
       COUNT(s.sale_id) AS total_orders,
       DENSE_RANK() OVER(PARTITION BY ci.city_name ORDER BY COUNT(s.sale_id) DESC) AS Rank
FROM sales AS s
JOIN products AS p
ON s.product_id=p.product_id
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON c.city_id=ci.city_id
GROUP BY 1,2
--ORDER BY 1,3 DESC
) AS t1
WHERE Rank<=3
```
7. **Customer Segmentation by City**  
   How many unique customers are there in each city who have purchased coffee products?
```sql
SELECT ci.city_name,
       COUNT(DISTINCT c.customer_id) AS unique_customers
FROM city AS ci
JOIN customers AS c
ON ci.city_id=c.city_id
JOIN sales AS s
ON s.customer_id=c.customer_id
WHERE s.product_id<=14
GROUP BY 1
```
8. **Average Sale vs Average Rent**  
   Find each city and their average sales per customer and avg rent per customer
```sql
SELECT ci.city_name, 
       ROUND(SUM(s.total)::numeric/COUNT(DISTINCT s.customer_id)::numeric,2) AS avg_sales_per_customer,
       ROUND(SUM(ci.estimated_rent)::numeric/COUNT(DISTINCT s.customer_id)::numeric,2) AS avg_rent_per_customer
FROM sales AS s
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON ci.city_id=c.city_id
GROUP BY ci.city_name
ORDER BY 2 DESC
```
9. **Monthly Sales Growth**  
     Calculate the percentage growth(or decline) in sales over different month by each city
```sql
WITH monthly_sales
AS 
(
SELECT ci.city_name,
       EXTRACT(MONTH FROM sale_date) AS month,
       EXTRACT(YEAR FROM sale_date) AS year,
       SUM(s.total) AS total_sales
FROM sales AS s
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON ci.city_id=c.city_id
GROUP BY 1,2,3
ORDER BY 1,3,2
),
growth_ratio AS
(SELECT city_name,
        month,
        year,
        total_sales AS current_month_sales,
        LAG(total_sales,1) OVER(PARTITION BY city_name ORDER BY year,month) AS last_month_sales
FROM monthly_sales
)
SELECT city_name,
       month,
       year,
       current_month_sales,
       last_month_sales,
       ROUND((current_month_sales-last_month_sales)::numeric/last_month_sales::numeric*100,2) AS growth_percentage
FROM growth_ratio
WHERE last_month_sales IS NOT NULL
```
10. **Market Potential Analysis**  
    Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated  coffee consumer
```sql

SELECT ci.city_name, 
       SUM(s.total) AS total_sales,
       SUM(ci.estimated_rent) AS total_rent,
       COUNT(DISTINCT s.customer_id) AS total_customer,
       ROUND((ci.population*0.25)/1000000,2) AS estimated_coffee_consumers_in_millions,
       ROUND(SUM(s.total)::numeric/COUNT(DISTINCT s.customer_id)::numeric,2) AS avg_sales_per_customer,
       ROUND(AVG(ci.estimated_rent)::numeric/COUNT(DISTINCT s.customer_id)::numeric,2) AS avg_rent_per_customer
FROM sales AS s
JOIN customers AS c
ON s.customer_id=c.customer_id
JOIN city AS ci
ON ci.city_id=c.city_id
GROUP BY ci.city_name, 5
ORDER BY 2 DESC

```
