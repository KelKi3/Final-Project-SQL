## Risk Areas:

The provided data presents several critical risk areas including the lack of context, missing values, duplicates, and inaccurate entries. These issues can significantly impact the reliability and usability of the data.

One of the challenges encountered during the data analysis process was the insufficient information available about the data's origin and collection methodology. The absence of proper context regarding the data's source makes it difficult to determine the accuracy, relevance, and appropriateness of the information. Having further knowledge of the source and purpose would make it easier to adhere to standard data formatting practices and to ensure that the data is in an appropriate and consistent form.

It would have been beneficial to have been able to allocate more time to the data preparation phase. Creating more streamlined and user-friendly versions of the tables would have facilitated a more effective assessment of the large number of duplicates, missing values, and the establishment of valid keys for relational databases.


## QA Process:

The QA procedure started with exploring and analysing the data for structure issues and quality issues. This does not include every query used, but examples of some that may have also been altered in order to view the data in other tables.

*Checking data types, ranges, missing data, outliers etc.*

```
SELECT column_name, data_type 
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'all_sessions';
```
```
SELECT MAX(time), MIN(time)
FROM all_sessions;
```
```
SELECT productsku, 'missing_price' AS reason
FROM all_sessions
WHERE productprice = 0 OR productprice IS NULL
```
```
SELECT productname, unitprice
FROM all_sessions al
JOIN analytics a ON al.visitid = a.visitid
WHERE UnitPrice = (SELECT MIN(UnitPrice) FROM analytics);
```
```
SELECT *
FROM products
WHERE sku IS NULL;
```

*Min, max, avg*
```
SELECT 'average unitprice',
ROUND(AVG(unitprice),2)
FROM analytics
UNION
SELECT 'minimum unitprice',
MIN(unitprice)
FROM analytics
UNION
SELECT 'maximum unitprice',
MAX(unitprice)
FROM analytics;
```

Under normal circumstances the data should be evaluated for accuracy against established quality measures, specified criteria or constraints. This is difficult to achieve for this project as there are no established QA measures set or previous data to compare to. Data can still be validated by ensuring proper formatting (ie dates, columns with predefined values contain only those values)
```
SELECT sku
FROM products
WHERE sku NOT LIKE 'GG%';
```
```
SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid NOT LIKE '0%'
ORDER BY fullvisitorid;
```
```
SELECT channelgrouping
FROM all_sessions
WHERE channelgrouping NOT IN ('Direct', 'Referral', 'Organic Search', 'Display', 'Paid Search', 'Affiliates');
```

Cleaning the data where necessary
- Full queries for the cleaning process can be seen in [cleaning_data.md](cleaning_data.md)
- identify data that can be removed or corrected
- check duplicates, formatting, data types

```
SELECT COUNT(DISTINCT sku)
FROM products;
```
```
SELECT *
FROM all_sessions
WHERE fullvisitorid IN(
	SELECT fullvisitorid
	FROM all_sessions
	GROUP BY fullvisitorid
	HAVING COUNT(fullvisitorid) > 1
	)
ORDER BY fullvisitorid;
```

Assess the integrity of data in relationship to other elements within the table or in other tables.

*Checking duplicates on primary keys*
```
SELECT
    fullvisitorid,
    COUNT(fullvisitorid)
FROM all_sessions
GROUP BY fullvisitorid
HAVING COUNT(fullvisitorid) > 1;
```


Check data for accuracy.
- cross reference data with external sources
- check for potential errors or anomalies
- look for and assess outlying data

*Shows how many rows per date*
```
SELECT DISTINCT * 
FROM (
	SELECT DISTINCT DATE(date) AS date, 
	COUNT(*) AS row_count 
	FROM all_sessions
WHERE date IS NOT NULL
GROUP BY date
ORDER BY date DESC
) AS date_count;
```
*Vs. average number of rows*
```
SELECT AVG(row_count) AS avg_num_date_rows
FROM (SELECT date, COUNT(*) AS row_count
	FROM all_sessions
	WHERE date IS NOT NULL 
	GROUP BY date
	)AS num_rows;
```

*Comparison of related data columns*
```
SELECT DISTINCT city, country
FROM all_sessions
WHERE city != '(not set)' AND country != '(not set)'
ORDER BY country, city;
```
*Viewing duplicate data*
```
SELECT al.productsku, al.productname
FROM all_sessions al
JOIN (SELECT productsku, COUNT(*)
	 FROM all_sessions
	 GROUP BY productsku
	 HAVING COUNT(*) >1) b
ON al.productsku = b.productsku
ORDER BY al.productsku
```
```
SELECT *
FROM products p
FULL OUTER JOIN sales_by_sku s 
ON p.sku = s.productsku
WHERE s.productsku IS NOT NULL AND p.sku IS NULL;
```
Will continue to monitor.