##Cleaning the Data

Upon initial view of the data it is clear that the data types will need to be adjusted to include appropriate data types and consistency across all the tables. 
```
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'all_sessions'
```
```
ALTER TABLE sales_by_sku
ALTER COLUMN total_ordered TYPE smallint
```
*To avoid inundating you with repetitive queries varying only by table and columns,  I will refrain from providing all the similar queries I used.*


There are no valid options for a primary key in the 'all_sessions' and 'analytics' tables due to duplicates.
- The most appropriate approach would involve breaking down the original tables into smaller tables containing related information and removing duplicates to identify unique values suitable for use as keys. However, considering the time constraints and the lack of context regarding the data in the tables, this process will not be conducted at this time.


*Checking for unique values*
```
SELECT DISTINCT fullvisitorid
FROM all_sessions
```
*Checking the data across the table where 'column' is a duplicate*
```
SELECT al.*
FROM all_sessions al
JOIN (SELECT fullvisitorid, COUNT(*)
	 FROM all_sessions
	 GROUP BY fullvisitorid
	 HAVING COUNT(*) >1) b
ON al.fullvisitorid = b.fullvisitorid
ORDER BY al.fullvisitorid
```

Considering that there are no unique keys for the 'all_sessions' and 'analytics' tables, the most suitable column to serve as a hypothetical primary key for the 'all_session' table would be the 'visitid' column. This assumption is based on the expectation that each visit to the site would have a distinct and unique value in the 'visitid' column.
For the analytics table a composite key between ‘fullvisitorid’ and ‘visitnumber’ could be made if moving the financial information into a separate table.

The ‘sku’ column is unique for the ‘products’ table and ‘productsku’ is unique for the ‘sales_report’ table and the ‘sales_by_sku’ table.
```
ALTER TABLE products
ADD PRIMARY KEY (sku)
```
The absence of context made it challenging to determine the appropriate data types for several columns.

- The 'time' column on the 'all_sessions' table gives no indication of units of measurement or what it is measuring. When attempting to convert this column to different data types, many returned errors. This suggests that the numbers in the column are possibly a measurement of time in milliseconds. This assumption is reinforced by the fact that converting the MAX value to longer units of time would result in a value representing over 800 hours.
- 'timeonsite' is assumed to be in seconds. If this column is tracking the time spent on the site, what is the other time column tracking?
- 'visitstarttime' contains date information as well time. When comparing the 'visitstarttime' and 'date columns to check if the date portion of the timestamp was redundent, there were inconsistent values found. All information was kept.
  
```
SELECT TO_TIMESTAMP(visitstarttime) AS formatted_datetime
FROM analytics
```
```
SELECT MAX(timeonsite), MIN(timeonsite)
FROM all_sessions
```
```
SELECT COUNT(*) AS null_count
FROM all_sessions
WHERE timeonsite IS NULL;
```

*Comparing the timeonsite to the epoch timestamp points to the timeonsite integer being a measure of seconds.*
```
SELECT timeonsite, TIMESTAMP 'epoch' + timeonsite * INTERVAL '1 second' AS formatted_datetime
FROM analytics;
```
```
SELECT visitstarttime,
       EXTRACT(YEAR FROM visitstarttime) AS year
FROM analytics
WHERE EXTRACT(YEAR FROM visitstarttime) != 2017
GROUP BY visitstarttime
```
```
SELECT EXTRACT(YEAR FROM date) AS year, COUNT(EXTRACT(YEAR FROM date))
FROM analytics
GROUP BY EXTRACT(YEAR FROM date)
```
```
SELECT EXTRACT(YEAR FROM date) AS year, COUNT(EXTRACT(YEAR FROM date))
FROM all_sessions
GROUP BY EXTRACT(YEAR FROM date)
```

Discrepancies with data entries, column titles, et al:
- Columns are checked for length, formatting and if the data is relevant and likely correct.
- Assessed columns for appropriate entries based on column title and surrounding information. 
- minimum and maximum values, averages, modes and NULL values

```
SELECT MAX(time), MIN(time)
FROM all_sessions
```
```
SELECT AVG(pageviews)
FROM analytics
```
```
SELECT totaltransactionrevenue
FROM all_sessions
WHERE totaltransactionrevenue IS NULL
```

- MIN and MAX values done on the 'unitssold' column in analytics shows a negative 'unitssold' value which is not possible.
- Format of sku numbers are not all uniform

*Pattern checking for sku fields*
```
SELECT sku
FROM products
WHERE sku NOT LIKE 'GG%'
```
```
SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid NOT LIKE '0%'
ORDER BY fullvisitorid
```
*Checking to see if any of the sku numbers that don't start with GG are inside any of the sku numbers that do.*
```
SELECT s1.productsku, s2.productsku
FROM sales_by_sku s1
JOIN sales_by_sku s2 ON s2.productsku = s1.productsku
WHERE s1.productsku NOT LIKE 'GG%' AND EXISTS (
	SELECT 1
	FROM sales_by_sku s2
	WHERE s2.productsku LIKE '%' || s1.productsku || '%'
	)
```
*Comparing sku values from different tables*
```
SELECT p.sku, s.productsku
FROM products p
FULL OUTER JOIN sales_by_sku s 
ON p.sku = s.productsku
```
*duplicate fullvisitorid data, checking length and formatting, comparisons to other tables*
```
SELECT MAX(length(CAST(fullvisitorid AS VARCHAR))), MIN(length(CAST(fullvisitorid AS VARCHAR)))
FROM analytics;
```
```
SELECT fullvisitorid
FROM all_sessions
WHERE REGEXP_LIKE(fullvisitorid, '[^0-9]')
```
```
SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid NOT LIKE '0%'
ORDER BY fullvisitorid
```
```
SELECT a.fullvisitorid AS full_id_analytics, al.fullvisitorid AS full_id_allsessions
FROM analytics a
LEFT JOIN all_sessions al 
ON (CAST(a.fullvisitorid AS VARCHAR)) = al.fullvisitorid
WHERE (CAST(a.fullvisitorid AS VARCHAR)) = al.fullvisitorid
--WHERE a.fullvisitorid IS NOT NULL AND al.fullvisitorid IS NOT NULL
ORDER BY al.fullvisitorid
```
Challenges in veryfying financial columns.
- We were given a hint that the “unitcost” column needed to be divided by 1,000,000. While some of the other monetary values in the database were large enough that it could be assumed the calculation would be required for them as well, there was no definitive indication for it without making further assumptions.
- One assumption I did make was that the hint in the instructions was referring to 'unitprice' column and not the 'productprice' column which has different values.

*Calculations on ‘unitprice’ columns*
```
SELECT ROUND(unitprice/1000000,2)
FROM analytics
```
```
SELECT MAX(unitprice/1000000), MIN(unitprice/1000000)
FROM analytics
```

- With a comparison of financial related data from two of the tables. The difference in ‘productprice’ vs ‘unitprice’  is noted.

```
SELECT ROUND(al.productprice/1000000,2), a.unitprice
FROM all_sessions al
JOIN analytics a
ON al.visitid = a.visitid
WHERE ROUND(al.productprice/1000000,2) != a.unitprice
```
- ‘transactionrevenue’ and ‘productrevenue’ have 4 values in it that are not NULL. Those 4 are equal to the corresponding values in 'totaltransactionrevenue', however 'totaltransactionrevenue' has many additional values that are not reflected in the other two columns.

*Comparison of data on financial columns*
```
SELECT transactionrevenue, totaltransactionrevenue, productrevenue
FROM all_sessions
WHERE transactionrevenue IS NOT NULL OR totaltransactionrevenue IS NOT NULL OR productrevenue IS NOT NULL
```
```
SELECT *
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL
```
```
SELECT al.transactions, al.productprice, a.unitprice, al.totaltransactionrevenue, al.transactionrevenue, al.productrevenue, a.revenue
FROM all_sessions al
JOIN analytics a ON al.visitid = a.visitid
WHERE totaltransactionrevenue IS NOT NULL
```
Missing values and NULLs in columns:
- There were many columns filled with only NULL values and no other data. As it is not clear if these columns will be used for data at a later date, they have been left in the tables.  
- 'userid' from the analytics table had potential to be a primary key if there were values in it.
- 'itemrevenue', 'searchkeyword', 'productrefundamount' only NULL values

```
SELECT userid
FROM analytics
WHERE userid IS NOT NULL
```

- Some missing values had ‘filler’ values. 
- Many values in the 'city' column were set as 'not available in demo dataset' or '(not set)'. 
  - Is it relevant to know that the information was unavailable verses not entered? 
- Comparisons were made to the 'country' column to see if there was information that could be added into 'city'.
- 24 values in the ‘country’ column are entered as ‘(not set)’
```
SELECT country, city
FROM all_sessions
WHERE country = '(not set)'
```

- NULL values for ‘timeonsite’ 
  - If the visitid is entered it would indicate that there should be some ‘timeonsite’ entered as well.
- 376 instances of the 'productprice' being set to 0 that have the same productnames as other products 
  - varying sku numbers and product prices.
- ‘Currencycode’ -all set to USD or NULL, regardless of related 'country'
```
SELECT DISTINCT currencycode, country
FROM all_sessions
WHERE currencycode IS NULL
```
After looking through the data columns individually, the columns from different tables were compared to help look for duplicate or missing data and to validate data.

- City, country combination incorrect. As entry has financial data attached, the city and country have been flagged for easier recognition and not removed from the database.
```
SELECT city
FROM all_sessions
WHERE city = 'New York' AND country = 'Canada'
```
```
UPDATE all_sessions
SET Country = 'Flagged Canada'
WHERE city = 'Flagged New York' AND country = 'Canada'
```

*Viewing data on itemssold with a negative value*
```
SELECT MIN(unitssold), MAX(unitssold)
FROM analytics
WHERE unitssold IS NOT NULL
```
```
SELECT *
FROM analytics
WHERE unitssold <0
```

*Check to see if stocklevels are less than the restockingleadtime*
```
SELECT stocklevel, restockingleadtime
FROM products
WHERE stocklevel < restockingleadtime
```
*Column data uniformity*
```
SELECT channelgrouping
FROM all_sessions
WHERE channelgrouping NOT IN ('Direct', 'Referral', 'Organic Search', 'Display', 'Paid Search', 'Affiliates')
```
*Distribution of entries and mode*
```
SELECT pageviews, COUNT(pageviews)
FROM analytics
GROUP BY pageviews
ORDER BY pageviews
LIMIT 1
```
```
SELECT DISTINCT ecommerceaction_type, COUNT(ecommerceaction_type)
FROM all_sessions
GROUP BY ecommerceaction_type
ORDER BY ecommerceaction_type
```
- Assessing 'sentimentscore', number of bad reviews. 
  - Negative values should not be possible.
```
SELECT COUNT(sentimentscore ) AS senitmentscore
FROM products
WHERE sentimentscore = 0.0
```
```
SELECT sentimentscore
FROM products
WHERE sentimentscore < 0
```
Duplicate values:

```
SELECT al.*
FROM all_sessions al
JOIN (SELECT productsku, COUNT(*)
	 FROM all_sessions
	 GROUP BY productsku
	 HAVING COUNT(*) >1) b
ON al.productsku = b.productsku
ORDER BY al.productsku
```

Productcategory:
- 74 distinct product categories
- formatting  "${escCatTitle}", redundant unnecessary wording ‘home’ in front of most of them
- some category names are ambiguous, - 757 ‘(not set)’
  - can categories be simplified to make them more effective?
  - added additional column with simplified categories?

```
SELECT DISTINCT productcategory
FROM all_sessions
WHERE productcategory LIKE 'Home%'
```
```
SELECT DISTINCT productcategory
FROM all_sessions
WHERE productcategory NOT LIKE 'Home%'
```

```
--Removed Home/ to try to consolidate categories

SELECT DISTINCT
	CASE
		WHEN productcategory LIKE 'Home%' THEN REPLACE(SUBSTRING(productcategory, 1, LENGTH(productcategory) - 1), 'Home/', '')
		WHEN productcategory LIKE '%/' THEN SUBSTRING(productcategory, 1, LENGTH(productcategory) - 1)
		ELSE productcategory
	END AS mod_productcategory
FROM all_sessions
ORDER BY mod_productcategory
```