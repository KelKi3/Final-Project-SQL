-Upon first view of the data it is clear that the data types will need to be adjusted to include appropriate types and consistency across the tables. 

SELECT column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'all_sessions'

ALTER TABLE sales_by_sku
ALTER COLUMN total_ordered TYPE smallint

(To avoid submitting a large amount of the same queries with variations by table and columns, I will not be providing all of the similar queries that I used.)


There are no valid options for a primary key in the 'all_sessions' and 'analytics' tables due to duplicates.
-The most appropriate action would be to break them down into smaller tables of related data and remove duplicates in order to find the unique values to use. However due to time constraints and lack of context regarding the data in the tables that process with not be administered at this time.


Checking for unique values

SELECT DISTINCT fullvisitorid
FROM all_sessions

Checking the data across the table where 'column' is a duplicate

SELECT al.*
FROM all_sessions al
JOIN (SELECT fullvisitorid, COUNT(*)
	 FROM all_sessions
	 GROUP BY fullvisitorid
	 HAVING COUNT(*) >1) b
ON al.fullvisitorid = b.fullvisitorid
ORDER BY al.fullvisitorid


Keeping in mind that there are no unique keys for those 2 tables, the most appropriate column to use as a hypothetical primary key for the 'all_session' table would be the 'visitid' column, as one could assume that there would be a unique value for each visit to the site. 
-For the analytics table a composite key between ‘fullvisitorid’ and ‘visitnumber’ could be made if moving the financial information into a separate table.

-The ‘sku’ column is unique for the ‘products’ table, ‘productsku’ for the ‘sales_report’ table and ‘productsku’ for the ‘sales_by_sku’ table

ALTER TABLE products
ADD PRIMARY KEY (sku)

-There was difficulty choosing the data in many columns due to the lack of context.
-The 'time' column on the all_sessions table gives no indication of units of measurement or what it is measuring. Many data types returned errors leading me to believe that the numbers in the column are possibly a measurement in milliseconds as conversions of the MAX number in longer units of time would equate to over 800 hours.
 -'timeonsite' assumed to be in seconds. If this column is tracking time spent on the site, what is the other time column tracking?
--'visitstarttime' contains date information as well time. Comparison was done to see if the date portion was redundent to the date column. But there were inconsistent values between them so all information was kept.

SELECT TO_TIMESTAMP(visitstarttime) AS formatted_datetime
FROM analytics

SELECT MAX(timeonsite), MIN(timeonsite)
FROM all_sessions

SELECT COUNT(*) AS null_count
FROM all_sessions
WHERE timeonsite IS NULL;

Looking at the 'timeonsite' column in an HH:MI:SS format to assess validity of the numbers

SELECT MAX(TO_CHAR((timeonsite||'seconds')::interval,'HH24:MI:SS')), 
MIN(TO_CHAR((timeonsite||'seconds')::interval,'HH24:MI:SS'))
FROM analytics

Comparing the timeonsite to the epoch timestamp  also points to the timeonsite integer being a measure of seconds.

SELECT timeonsite, TIMESTAMP 'epoch' + timeonsite * INTERVAL '1 second' AS formatted_datetime
FROM analytics;

SELECT visitstarttime
FROM analytics
WHERE visitstarttime IS NULL

SELECT visitstarttime,
       EXTRACT(YEAR FROM visitstarttime) AS year
FROM analytics
WHERE EXTRACT(YEAR FROM visitstarttime) != 2017
GROUP BY visitstarttime

SELECT EXTRACT(YEAR FROM date) AS month, COUNT(EXTRACT(YEAR FROM date))
FROM analytics
GROUP BY EXTRACT(YEAR FROM date)

The date column on the all_sessions table includes dates in 2016.

SELECT EXTRACT(YEAR FROM date) AS month, COUNT(EXTRACT(YEAR FROM date))
FROM all_sessions
GROUP BY EXTRACT(YEAR FROM date)

Other assessments on time columns

SELECT MAX(time), MIN(time)
FROM all_sessions

SELECT time, 
	ROUND(CAST(time AS NUMERIC)/1000, 2) AS seconds
FROM all_sessions




Data entries based on column title and surrounding information are not all valid:
-Columns are checked for length, formatting and if the data is relevant and likely correct.
-Assessed columns for appropriate entries based on column title and surrounding information. 
- minimum and maximum values, averages, modes and NULL values

MIN and MAX values

SELECT MAX(time), MIN(time)
FROM all_sessions

Calculating average

SELECT AVG(pageviews)
FROM analytics

Checking for NULL values and viewing the values that are not NULL

SELECT totaltransactionrevenue
FROM all_sessions
WHERE totaltransactionrevenue IS NULL



Min and max values done on the 'unitssold' column in analytics showed that there is a negative value for one row which is not possible.

-Format of sku numbers are not all uniform

Pattern checking for sku fields

SELECT sku
FROM products
WHERE sku NOT LIKE 'GG%'

SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid NOT LIKE '0%'
ORDER BY fullvisitorid

Checking to see if any of the sku numbers that don't start with GG are inside any of the sku numbers that do.

SELECT s1.productsku, s2.productsku
FROM sales_by_sku s1
JOIN sales_by_sku s2 ON s2.productsku = s1.productsku
WHERE s1.productsku NOT LIKE 'GG%' AND EXISTS (
	SELECT 1
	FROM sales_by_sku s2
	WHERE s2.productsku LIKE '%' || s1.productsku || '%'

Comparing sku values from different tables

SELECT p.sku, s.productsku
FROM products p
FULL OUTER JOIN sales_by_sku s 
ON p.sku = s.productsku



-duplicate fullvisitorid data, checking length and formatting, compare to analysis table

SELECT DISTINCT fullvisitorid
FROM all_sessions

SELECT MAX(length(CAST(fullvisitorid AS VARCHAR))), MIN(length(CAST(fullvisitorid AS VARCHAR)))
FROM analytics;

SELECT MAX(length(CAST(fullvisitorid AS VARCHAR))), MIN(length(CAST(fullvisitorid AS VARCHAR)))
FROM all_sessions;

SELECT fullvisitorid
FROM all_sessions
WHERE REGEXP_LIKE(fullvisitorid, '[^0-9]')

SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid NOT LIKE '0%'
ORDER BY fullvisitorid

SELECT a.fullvisitorid AS fvi_analytics, al.fullvisitorid AS fvi_allsessions
FROM analytics a
LEFT JOIN all_sessions al 
ON (CAST(a.fullvisitorid AS VARCHAR)) = al.fullvisitorid
WHERE (CAST(a.fullvisitorid AS VARCHAR)) = al.fullvisitorid
--WHERE a.fullvisitorid IS NOT NULL AND al.fullvisitorid IS NOT NULL
ORDER BY al.fullvisitorid

-It was difficult to varify whether or not the financial columns had valid information or not. 
-We were given a hint that the “unitcost” column needed to be divided by 1,000,000. While some of the other monetary values in the database were large enough that it could be assumed the calculation would be required for them as well, there was no definitive indication for it without making further assumptions.
-One assumption I did make was that the hint in the instructions was referring to 'unitprice' column and not the 'productprice' column which have different values from each other.

Calculations on ‘unitprice’ columns

SELECT unitprice/1000000
FROM analytics

SELECT MAX(unitprice/1000000), MIN(unitprice/1000000)
FROM analytics


-With a comparison of financial related data from two of the tables. The difference in ‘productprice’ vs ‘unitprice’ even if calculations were done on both columns is noted.
-Productrefundamount and itemrevenue columns had no data

SELECT ROUND(al.productprice/1000000,2), a.unitprice
FROM all_sessions al
JOIN analytics a
ON al.visitid = a.visitid
WHERE ROUND(al.productprice/1000000,2) != a.unitprice

-There were many columns filled with only NULL values and no other data. As it is not clear if these columns will be used for data at a later date, they have been left in the tables.  
-'userid' from the analytics table had potential to be a primary key if there were values in it.
-itemquantity only NULL. What is the difference between this and productquantity?
-item revenue only NULL
-searchkeyword only NULL
- Product refund amount only NULL

SELECT userid
FROM analytics
WHERE userid IS NOT NULL

Missing values in many columns:
-Missing values that were not NULL and filled in with other ‘filler’ values. 
-Many values in the 'city' column were set as 'not available in demo dataset' and some were '(not set)'. Is it relevant to know that the information was unavailable verses not entered? Comparisons were made to the country column to see if there was information that could be added in.
-24 values in the ‘country’ column are entered as ‘(not set)’

SELECT country, city
FROM all_sessions
WHERE country = '(not set)'

-City, country combination incorrect. As entry has financial data attached, the city and country have been flagged for easier recognition

SELECT city
FROM all_sessions
WHERE city = 'New York' AND country = 'Canada'

UPDATE all_sessions
SET Country = 'Flagged Canada'
WHERE city = 'Flagged New York' AND country = 'Canada'

-NULL values for ‘timeonsite’ If the visitid is entered it would indicate that there should be some ‘timeonsite’ entered as well

-376 instances of the productprice being set to 0 that have the same productnames as other products, carrying sku numbers and product prices.

- ‘Currencycode’ -all set to USD except for 3

SELECT DISTINCT currencycode, country
FROM all_sessions
WHERE currencycode IS NULL


-‘transactionrevenue’ and ‘productrevenue’ have 4 values in it that are not NULL. Those 4 are equal to the corresponding values in totaltransactionrevenue, however totaltransactionrevenue has many additional values in the column

Comparison of data on financial columns

SELECT transactionrevenue, totaltransactionrevenue, productrevenue
FROM all_sessions
WHERE transactionrevenue IS NOT NULL OR totaltransactionrevenue IS NOT NULL OR productrevenue IS NOT NULL

SELECT *
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL

SELECT al.transactions, al.productprice, a.unitprice, al.totaltransactionrevenue, al.transactionrevenue, al.productrevenue, a.revenue
FROM all_sessions al
JOIN analytics a ON al.visitid = a.visitid
WHERE totaltransactionrevenue IS NOT NULL

¬-not obvious what the ratio column is comparing

-After looking through the data columns individually, the columns from different tables were compared to help look for duplicate or missing data and to check for invalid data.

Viewing data on itemssold with a negative value

SELECT MIN(unitssold), MAX(unitssold)
FROM analytics
WHERE unitssold IS NOT NULL

SELECT *
FROM analytics
WHERE unitssold <0


Check to see if stocklevels are less than the restockingleadtime

SELECT stocklevel, restockingleadtime
FROM products
WHERE stocklevel < restockingleadtime

Is all the data uniform in the column

SELECT channelgrouping
FROM all_sessions
WHERE channelgrouping NOT IN ('Direct', 'Referral', 'Organic Search', 'Display', 'Paid Search', 'Affiliates')

Looking at the distribution of entries and mode

SELECT pageviews, COUNT(pageviews)
FROM analytics
GROUP BY pageviews
ORDER BY pageviews
LIMIT 1

SELECT DISTINCT ecommerceaction_type, COUNT(ecommerceaction_type)
FROM all_sessions
GROUP BY ecommerceaction_type
ORDER BY ecommerceaction_type

All years from ‘visitstarttime’ and ‘date’ on the analytics table are 2017
The date column on the all_sessions table includes dates in 2016.


SELECT EXTRACT(YEAR FROM date) AS month, COUNT(EXTRACT(YEAR FROM date))
FROM analytics
GROUP BY EXTRACT(YEAR FROM date)

SELECT EXTRACT(MONTH FROM date) AS month, COUNT(EXTRACT(MONTH FROM date))
FROM analytics
GROUP BY EXTRACT(MONTH FROM date)

SELECT EXTRACT(YEAR FROM date) AS month, COUNT(EXTRACT(YEAR FROM date))
FROM all_sessions
GROUP BY EXTRACT(YEAR FROM date)

Assessing sentiment score. Number of bad reviews. Negative values should not be possible.

SELECT COUNT(sentimentscore )
FROM products
WHERE sentimentscore = 0.0

SELECT MIN(sentimentscore), MAX(sentimentscore)
FROM products

SELECT sentimentscore
FROM products
WHERE sentimentscore < 0

Duplicate values:
-Large number of duplicates in the product table with unique sku numbers and same productname
-duplicates in visitid column, would be a good candidate for primary key if duplicates could be removed

SELECT al.*
FROM all_sessions al
JOIN (SELECT productsku, COUNT(*)
	 FROM all_sessions
	 GROUP BY productsku
	 HAVING COUNT(*) >1) b
ON al.productsku = b.productsku
ORDER BY al.productsku

Productcategory:
-74 distinct product categories
-formatting  "${escCatTitle}", redundant unnecessary wording ‘home’ in front of most of them
-some category names are ambiguous
-757 ‘(not set)’
-can they be simplified to make more effective categories?
-add additional column with simplified categories?

SELECT DISTINCT productcategory
FROM all_sessions

SELECT DISTINCT productcategory
FROM all_sessions
WHERE productcategory LIKE 'Home%'

SELECT DISTINCT productcategory
FROM all_sessions
WHERE productcategory NOT LIKE 'Home%'

Removed Home/ to try to consolidate categories

SELECT DISTINCT
	CASE
		WHEN productcategory LIKE 'Home%' THEN REPLACE(SUBSTRING(productcategory, 1, LENGTH(productcategory) - 1), 'Home/', '')
		WHEN productcategory LIKE '%/' THEN SUBSTRING(productcategory, 1, LENGTH(productcategory) - 1)
		ELSE productcategory
	END AS mod_productcategory
FROM all_sessions
ORDER BY mod_productcategory


