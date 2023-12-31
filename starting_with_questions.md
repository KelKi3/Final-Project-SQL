

# Starting With Questions

    
### Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

**SQL Queries:**

There is a city and country combination of New York and Canada. There is no way to tell which is the correct one. The revenue data attached to it can still be useful for situations where city and country do not matter. So the city and country of that row will be altered for easier clarification.
```
UPDATE all_sessions
SET Country = 'Flagged Canada'
WHERE city = 'Flagged New York' AND country = 'Canada'
```
*total transaction revenue by country*
```
SELECT country, SUM(totaltransactionrevenue/1000000) AS total_transaction_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND
city != 'Flagged New York'
GROUP BY country
ORDER BY total_transaction_revenue DESC
```

*total transaction revenue by city*
```
SELECT city,  SUM(totaltransactionrevenue/1000000) AS total_transaction_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND
city != 'Flagged New York' AND
city != '(not set)'
GROUP BY city
ORDER BY total_transaction_revenue DESC
```

**Answer:**

The highest level of transaction revenue per country is seen in the United States.

Determining the highest transaction revenue per city is more difficult. The largest number of transactions is in the city category ‘(not set)’.  Majority of the cities that are (not set) are located in the USA, but there are a significant amount in other countries as well, such as United Kingdom, Canada and India. In total there are 135 countries that have cities that have no indication of what they are.

Eliminating the indeterminant ‘(not set)’ cities from our data. The city with the highest total transaction revenue is San Francisco (USA).


### Question 2: What is the average number of products ordered from visitors in each city and country?**


**SQL Queries:**

*Much time was spent comparing the tables trying to figure out which column to use for the 'number of products'. After finding that there was no clear, logical choice, I have decided to just choose the 'unitssold' column to use to demonstrate my solution for this task and tasks 3 and 4. The queries immediately following are what I used in the comparison of the different options for 'number of product'.*

*Comparing sku and order combinations between tables:*
```
SELECT sr.productsku AS sr_sku, 
	sr.totalordered AS sr_orders, 
	sbs.productsku AS sbs_sku, 
	sbs.total_ordered AS sbs_orders, 
	p.sku AS p_sku, 
	p.orderedquantity AS p_orders
FROM sales_by_sku sbs
FULL OUTER JOIN sales_report sr ON sbs.productsku = sr.productsku
FULL OUTER JOIN products p ON p.sku = sr.productsku
```
- 'orderedquantity' on the 'products' table are vastly different from the quantities on the 'sales_by_sku' and 'sales_report' tables. 
  - My assumption is that these are the orders made by the site in order to have product to sell to the visitors as it is not likely that the average person bought 15170 Kick Balls. Will not use this information for these tasks.

- 'productsku' and 'total_ordered' from 'sales_by_sku' matches 'sales_report', so to use only one table is appropriate.     
  - There are extra 'productsku' on the 'all_sessions' table, but there are no products information for them. 

- Will use 'unitssold' from the 'analytics' table. 
  - Assuming that if there is a 'unitssold' value, that there has been a unit sold regardless of the 'totaltransactionrevenue' value.
```
SELECT a.unitssold, al.productsku, productquantity, totaltransactionrevenue, productprice
FROM analytics a
JOIN all_sessions al ON a.visitid = al.visitid
WHERE unitssold IS NOT NULL
```
```
SELECT sr.productsku, sr.totalordered, al.productsku, productquantity, totaltransactionrevenue, productprice
FROM sales_report sr
JOIN all_sessions al ON sr.productsku = al.productsku
WHERE sr.totalordered != 0
```
**Solution Queries**

*Shows the average number of products ordered per country*
```
SELECT al.country, ROUND(AVG(a.unitssold), 2) AS avg_units_sold
FROM analytics a
JOIN all_sessions al 
ON a.visitid = al.visitid
WHERE city != 'Flagged New York'
GROUP BY  al.country
HAVING AVG(a.unitssold) > 0
ORDER BY AVG(a.unitssold) DESC
```
*Shows the average number of products ordered per city*
```
SELECT al.city, ROUND(AVG(a.unitssold),2) AS avg_units_sold
FROM analytics a
JOIN all_sessions al 
ON a.visitid = al.visitid
WHERE city != '(not set)'  AND city != 'Flagged New York'
GROUP BY al.city
HAVING AVG(a.unitssold) > 0
ORDER BY avg_units_sold DESC
```
**Answer:**

As it is not effcient to write out all the averages for every single city and country I will list the highest and lowest average for each and show an immage of the 5 highest by average products sold.


For the average number of products ordered in each country, the highest is from Canada at 2.44 and lowest is Denmark at 1.

![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/d3297231-2db2-4a83-b3cc-db68eb458195)


For the average number of products ordered in each city, the highest is from Chicago at 5  and lowest is Hong Kong at 1.

![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/d4d89d64-3ea8-47ba-804e-c5721fca2127)


### Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


**SQL Queries:**

Due to the lack of organization and clarity in the product categories, I have created a new column called 'modifiedcategories' to help with searching through the products. This array column will make it easier to perform searches based on broader categories. However, it is still possible to conduct more detailed searches using the productcategories and productname columns when necessary.
```
ALTER TABLE all_sessions
ADD modifiedcategories VARCHAR(100);
```
*Change column into array to use various categories per item*
```
ALTER TABLE all_sessions
ALTER COLUMN modifiedcategories TYPE varchar(255)[] USING ARRAY[modifiedcategories];];
```
*example query for filling the new column*
```
UPDATE all_sessions 
SET modifiedcategories = modifiedcategories || '{Men''s}'
WHERE productname LIKE '%Men''s%'
SELECT *
FROM all_sessions
WHERE 'Men''s' = ANY(modifiedcategories)
```
*Created 25 simplified categories*
```
SELECT DISTINCT unnest(modifiedcategories) AS unique_value
FROM all_sessions;
```
![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/b5c52e22-49e1-4331-9e5c-0f055468bfa2)

**Solution Queries**

*Showing how many items sold under certain catergories per country*
```
SELECT al.country, SUM(a.unitssold) AS unitssold, al.productcategory, al.modifiedcategories
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL 
GROUP BY al.country, al.productcategory, al.modifiedcategories
ORDER BY country, unitssold DESC
```
*Showing how many items sold under certain catergories per city*
```
SELECT al.city, SUM(a.unitssold) AS unitssold, al.productcategory, al.modifiedcategories
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND city != '(not set)'
GROUP BY al.city, al.productcategory, al.modifiedcategories
ORDER BY city, unitssold DESC
```
Examples of queries for pattern checking:

*Products ordered under certain categories from United States*
```
SELECT SUM(a.unitssold) AS unitssold, al.productcategory
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND country = 'United States' AND productcategory != '(not set)'
GROUP BY  al.productcategory
ORDER BY unitssold DESC
```
*Total apparel products sold in the United States*
```
SELECT SUM(a.unitssold) AS unitssold
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND country = 'United States' AND productcategory LIKE '%Apparel%'
ORDER BY unitssold DESC
```
*Showing how many unitssold that fall under specific modified categories*

```
SELECT country, sum(unitssold) AS units_sold_by_category
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE ARRAY['Apparel']::character varying[] && modifiedcategories AND unitssold IS NOT NULL
GROUP BY country
ORDER BY sum(unitssold) DESC
```
*The most unitssold per city per category*
```
SELECT city, unitssold, productcategory, modifiedcategories
FROM (
    SELECT 
        al.city, SUM(a.unitssold) AS unitssold, al.productcategory, al.modifiedcategories,
        RANK() OVER (PARTITION BY al.city ORDER BY SUM(a.unitssold) DESC) AS rank
    FROM all_sessions al
    JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
    WHERE a.unitssold IS NOT NULL AND city != '(not set)'
    GROUP BY al.city, al.productcategory, al.modifiedcategories
) ranked_data
WHERE rank = 1
ORDER BY city;
```
*Filtered to cities in the United States*
```
SELECT al.city, SUM(a.unitssold) AS unitssold, al.productcategory, al.modifiedcategories
FROM all_sessions al
JOIN analytics a ON al.fullvisitorid = a.fullvisitorid
WHERE a.unitssold IS NOT NULL AND city != '(not set)' AND country = 'United States'
GROUP BY al.city, al.productcategory, al.modifiedcategories
ORDER BY unitssold DESC
```

Answer:

Observations regarding product categories and countries:
- Apparel is the category with the highest diversity of countries, with 28 different countries purchasing these items. Among them, the United States stands out as the primary purchaser of apparel products.
- The United States is the only country with purchases in all the different product categories.

Observations regarding product categories and city:
-Apparel also emerges as the category with the most diverse range of cities purchasing these items.
-the product categories with the highest purchases rate per city in the USA are Stickers from Mountain View, Bags from New York and YouTube waterbottle from San Bruno




### Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?


**SQL Queries:**

*Top selling product per country*
```
WITH totals AS (
SELECT al.country, al.productname, SUM(a.unitssold) AS unitssold, al.modifiedcategories
FROM all_sessions al
JOIN analytics a ON al.visitid = a.visitid
WHERE a.unitssold IS NOT NULL
GROUP BY al.country, al.productname, al.modifiedcategories
)

SELECT t.country, t.productname, t.unitssold, t.modifiedcategories
FROM totals t
JOIN (
    SELECT country, MAX(unitssold) AS max_unitsold
    FROM totals
    GROUP BY country
) max_t ON t.country = max_t.country AND t.unitssold = max_t.max_unitsold
ORDER BY t.unitssold DESC;
```


*Top selling product per city*
```
WITH totals AS (
    SELECT al.country, al.city, al.productname, SUM(a.unitssold) AS unitssold, al.modifiedcategories
    FROM all_sessions al
    JOIN analytics a ON al.visitid = a.visitid
    WHERE a.unitssold IS NOT NULL AND city != '(not set)'
    GROUP BY al.country, al.city, al.productname, al.modifiedcategories
)

SELECT t.city, t.productname, t.unitssold, t.modifiedcategories
FROM totals t
JOIN (
    SELECT city, MAX(unitssold) AS max_unitsold
    FROM totals
    GROUP BY city
) max_t ON t.city = max_t.city AND t.unitssold = max_t.max_unitsold
ORDER BY t.unitssold DESC;
```
**Answer:**

The products with the top three sales by country are Brand related products with Google Alpine Style Backpack from United States, Waze Pack of 9 Decal Set from Canada and Android RFID Journal from Egypt.

The products with the top three sales by city are all from United States; "SPF-15 Slim & Slender Lip Balm" from Sunnyvale and the "Google Alpine Style Backpack" from New York and Chicago


### Question 5: Can we summarize the impact of revenue generated from each city/country?**

**SQL Queries:**

```
SELECT city, country,
    SUM(totaltransactionrevenue) AS total_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND country != 'Flagged Canada' AND city != '(not set)'
GROUP BY city, country
ORDER BY total_revenue DESC;
```
```
SELECT country,
    SUM(totaltransactionrevenue) AS total_revenue
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL AND country != 'Flagged Canada'
GROUP BY country
ORDER BY total_revenue DESC;
```

**Answer:**

The amount of revenue can vary significantly depending on many things, such as the size of the country and their level of development.

The previous queries demonstrate that consumer spending is notably higher in the USA. Several factors can contribute to this trend, including a large population in a developed country with access to technology and online spending capabilities. These conditions create significant potential for higher revenue generation in the USA market.
