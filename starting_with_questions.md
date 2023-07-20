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

Productsku and total_ordered from sales_by_sku matches sales_report, so to use only one table is appropriate. There are extra productsku on the all_sessions table, but there are no products ordered information for them. 

Will use unitssold from the analytics total_ordered for this task and tasks 3 and 4 for consistency. Also are assuming that because there is a unitssold value, that there was a unit sold regardless of total transaction revenue.

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

SELECT al.country, ROUND(AVG(a.unitssold), 2) AS avg_unit_ssold
FROM analytics a
JOIN all_sessions al 
ON a.visitid = al.visitid
WHERE city != 'Flagged New York'
GROUP BY  al.country
HAVING AVG(a.unitssold) > 0
ORDER BY AVG(a.unitssold) DESC

Shows the average number of products ordered per city

SELECT al.city, ROUND(AVG(a.unitssold),2) AS aveproductsordered
FROM analytics a
JOIN all_sessions al 
ON a.visitid = al.visitid
WHERE city != '(not set)'  AND city != 'Flagged New York'
GROUP BY al.city
HAVING AVG(a.unitssold) > 0
ORDER BY aveproductsordered DESC

Answer:

As it is not effcient to write out all the averages for every single city and country I will list the highest and lowest average for each.


For the average number of products ordered in each country, the highest is from Canada at 2.44 and lowest is Denmark at 1.

For the average number of products ordered in each city, the highest is from Chicago at 5  and lowest is Hong Kong at 1.









**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

Will also be using u

It is incredibly difficult to discern a pattern from the product categories in relation to anything as the labelling is not organized and is very confusing. The labels that are present are sometimes vague or missing and some are not appropriately descriptive. There are also products that could be categorized under many different categories.  For this reason, I have created a column of simplified categories in an array to help with searching through the products. It is still possible to do a more detailed search through the productcategories and productname columns it needed.

ALTER TABLE all_sessions
ADD modifiedcategories VARCHAR(100);

Change column into array to use various categories per item

ALTER TABLE all_sessions
ALTER COLUMN modifiedcategories TYPE varchar(255)[] USING ARRAY[modifiedcategories];];

example query for filling the new column

UPDATE all_sessions 
SET modifiedcategories = modifiedcategories || '{Men''s}'
WHERE productname LIKE '%Men''s%'
SELECT *
FROM all_sessions
WHERE 'Men''s' = ANY(modifiedcategories)

Created 25 simplified categories 

SELECT DISTINCT unnest(modifiedcategories) AS unique_value
FROM all_sessions;

![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/b5c52e22-49e1-4331-9e5c-0f055468bfa2)

Showing how many items sold under certain catergories per country

SELECT al.country, SUM(a.unitssold) AS unitssold, al.productcategory, al.modifiedcategories
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL 
GROUP BY al.country, al.productcategory, al.modifiedcategories
ORDER BY country, unitssold DESC

Examples of queries for pattern checking

Products ordered under certain categories from United States

SELECT SUM(a.unitssold) AS unitssold, al.productcategory
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND country = 'United States' AND productcategory != '(not set)'
GROUP BY  al.productcategory
ORDER BY unitssold DESC

Total apparel products sold in the United States

SELECT SUM(a.unitssold) AS unitssold
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND country = 'United States' AND productcategory LIKE '%Apparel%'
ORDER BY unitssold DESC

Showing how many unitssold that fall under specific modified categories

Apparel

SELECT country, sum(unitssold) AS units_sold_by_category
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE ARRAY['Apparel']::character varying[] && modifiedcategories AND unitssold IS NOT NULL
GROUP BY country
ORDER BY sum(unitssold) DESC


Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







