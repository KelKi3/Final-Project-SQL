Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

There is a city and country combination of New York and Canada. There is no way to tell which is the correct one.
The revenue data attached to it can still be useful for situations where city and country do not matter. So the city and country of that row will be altered for easier clarification.

UPDATE all_sessions
SET Country = 'Flagged Canada'
WHERE city = 'Flagged New York' AND country = 'Canada'


total transaction revenue by country

SELECT country, SUM(totaltransactionrevenue/1000000) AS total_transaction_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND
city != 'Flagged New York'
GROUP BY country
ORDER BY total_transaction_revenue DESC


total transaction revenue by city

SELECT city,  SUM(totaltransactionrevenue/1000000) AS total_transaction_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND
city != 'Flagged New York' AND
city != '(not set)'
GROUP BY city
ORDER BY total_transaction_revenue DESC


Answer:

The highest level of transaction revenues per country is seen in the United States.

Determining the highest transaction revenue per city is more difficult. 
The largest count of transactions is for the city category ‘(not set)’.  Majority of the cities that are (not set) are located in the USA, but there are a significant amount in other countries such as United Kingdom, Canada and India. In total there are 135 countries that have cities that have no indication of what they are.

Eliminating the indeterminant ‘(not set)’ cities from our data. The city with the highest total transaction revenue is San Francisco (USA).







**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:

AFter much comparison between the tables trying to figure out which colum to use for the 'number of products' for this question, I have decided to just choose a column to use to demonstrate the queries used to come up with a solution to this task. The queries immediately following are what I used in the comparison of the different options.

Comparing sku and order combinations between tables:

SELECT sr.productsku AS sr_sku, 
	sr.totalordered AS sr_orders, 
	sbs.productsku AS sbs_sku, 
	sbs.total_ordered AS sbs_orders, 
	p.sku AS p_sku, 
	p.orderedquantity AS p_orders
FROM sales_by_sku sbs
FULL OUTER JOIN sales_report sr ON sbs.productsku = sr.productsku
FULL OUTER JOIN products p ON p.sku = sr.productsku

Product orders are vastly different from the order values per sku on the sales_by_sku and sales_report tables. Assumption that these are the orders made by the site in order to have product to sell to the visitors as it is not likely that the average person bought 15170 Kick Balls. Will not use this information for these tasks.

Productsku and total_ordered from sales_by_sku matches sales_report, so to use only one table is appropriate. There are extra productsku on the all_sessions table, but there are no products ordered information for them. Will use sales_report total_ordered for this task.

SELECT a.unitssold, al.productsku, productquantity, totaltransactionrevenue, productprice
FROM analytics a
JOIN all_sessions al ON a.visitid = al.visitid
WHERE unitssold IS NOT NULL

SELECT sr.productsku, sr.totalordered, al.productsku, productquantity, totaltransactionrevenue, productprice
FROM sales_report sr
JOIN all_sessions al ON sr.productsku = al.productsku
WHERE sr.totalordered != 0

SELECT *
FROM sales_by_sku sbs
FULL OUTER JOIN sales_report sr ON sbs.productsku = sr.productsku

Solution Queries

Shows the average number of products ordered per country

SELECT al.country, ROUND(AVG(sr.totalordered), 2) AS avgproductsordered
FROM sales_report sr
JOIN all_sessions al 
ON sr.productsku = al.productsku
GROUP BY  al.country
HAVING AVG(sr.totalordered) > 0
ORDER BY AVG(sr.totalordered) DESC

Shows the average number of products ordered per city

SELECT al.city, ROUND(AVG(sr.totalordered),2) AS aveproductsordered
FROM sales_report sr
JOIN all_sessions al 
ON sr.productsku = al.productsku
WHERE city != '(not set)'  AND city != 'Flagged New York'
GROUP BY al.city
HAVING AVG(sr.totalordered) > 0
ORDER BY aveproductsordered DESC

Answer:

As it is not effcient to write out all the averages for every single city and country I will list the highest and lowest average for each.


For the average number of products ordered in each country, the highest is from Saudi Arabia at 96.29 and lowest is Irag at 0.50.

For the average number of products ordered in each city, the highest is from Riyadh at 319 (although they only have one order) and lowest is Kansas City at 0.67.




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







