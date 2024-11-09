# Helping SRI Coffee to choose their physical store locations
## Objective
**The goal of this project is to analyze the sales data of SRI Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening their physical shops based on consumer demand and sales performance.**

## Recommendations
After analyzing the data, the recommended top three cities for new store openings are:

**City 1: Pune**  
1)Highest total revenue generator. 
2)Low average rent per customer.
3)Average sales per customer is very high.

**City 2: Delhi**  
1)Highest estimated coffee consumers. 
2)2nd highest unique customers.
3)Average rent per customer is 330 (which is under 500).

**City 3: Jaipur**  
1)Highest unique customers. 
2)Lowest average rent per customer.
3)Average rent per customer is 330 (which is under 500).

![Company Logo](https://github.com/TanvirRaihanKhan/Helping-a-Coffee-Shop-to-expand-their-business-with-help-of-SQL/blob/main/cofee_store.jpg)

## Key questions to be answered before picking a city 
1. **Coffee Consumers Count**  
   How many people in each city are estimated to consume coffee, given that 25% of the population does?
```sql
SELECT city_name, ROUND((population*0.25)/1000000,2) AS cofee_consumers_in_millions, city_rank
FROM city
ORDER BY 2 DESC;
```
   


