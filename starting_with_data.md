Question 1: 

What is the most common way visitors are finding their way to the site?

https://evenbound.com/blog/direct-traffic-vs-organic-traffic

Direct traffic – not directed from a referring site, traffic from an unknown source

Organic traffic – comes to your site from a search engine that isn’t paid for, 
	-indication of inbound marketing

Referral - Traffic that comes to your website from any other website that is not a social media platform or a search engine. Someone clicking over to your website from a hyperlink on another blog would be considered referral traffic.

Paid search - Paid search traffic is any traffic that comes from a paid search campaign you've launched on a search engine like Google or Bing.

Affiliate – traffic (visits, clicks) delivered by affiliates to your digital property

Display - visitors that come to a website as a result of clicking display ads on other websites, apps, or platforms.


SELECT channelgrouping, COUNT(channelgrouping) AS visitcount
FROM all_sessions
GROUP BY channelgrouping
ORDER BY visitcount DESC

Showing that the bulk of the visits come from organic searches of a search engine 

Analytics

553 entries with repeated visit id

SELECT visitid, COUNT (visitid)
FROM all_sessions
GROUP BY visitid
HAVING COUNT(visitid) > 1

Checking info in the duplicate visitorids

SELECT *
FROM all_sessions
WHERE visitid IN(
	SELECT visitid
	FROM all_sessions
	GROUP BY visitid
	HAVING COUNT(visitid) > 1
	)
ORDER BY visited



Question 2: 

Viewing breakdown of the number of visits per each month of the year, assuming we had a proper primary key and the visitid’s were actually unique

SELECT COUNT(visitid)AS numberofvisits,
	EXTRACT(year from (DATE_TRUNC('month', date))) AS year, 
	EXTRACT(month from (DATE_TRUNC('month', date)))AS month
FROM all_sessions
GROUP BY year, month
ORDER BY year, month

 




Question 3: 

Average time on site for visitors with transactions vs without

SELECT ROUND(AVG(a.timeonsite)/60,0) AS avg_time_min
FROM analytics a
JOIN all_sessions al ON al.visitid = a.visitid
WHERE transactions IS NOT NULL

SELECT ROUND(AVG(a.timeonsite)/60,0) AS avg_time_min
FROM analytics a
JOIN all_sessions al ON al.visitid = a.visitid
WHERE transactions IS NULL



Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
