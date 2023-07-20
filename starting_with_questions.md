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

The highest level of transaction revenues per country is seen in the United States and per city is San Fransciso.


**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







