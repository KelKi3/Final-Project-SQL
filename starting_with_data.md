
## Question 1: 

### What is the most common way visitors are finding their way to the site?

```
SELECT DISTINCT channelgrouping
FROM all_sessions
```

```
SELECT channelgrouping, COUNT(channelgrouping) AS visitcount
FROM all_sessions
GROUP BY channelgrouping
ORDER BY visitcount DESC
```

The query's findings show that the website receives a significant portion of its visits from organic searches conducted on search engines. This information provides valuable insights to the business regarding how their visitors are discovering their website. It helps in determining the effectiveness of various traffic channels and can aid the business in prioritizing successful search strategies and reevaluating less effective traffic ones.


## Question 2: 

###In what months each year have the most number of visits?

```
SELECT COUNT(visitid)AS numberofvisits,
	EXTRACT(year from (DATE_TRUNC('month', date))) AS year, 
	EXTRACT(month from (DATE_TRUNC('month', date)))AS month
FROM all_sessions
GROUP BY year, month
ORDER BY year, month
```
```
SELECT 
    RANK() OVER (PARTITION BY EXTRACT(year from (DATE_TRUNC('month', date))) 
				 ORDER BY COUNT(visitid) DESC) AS visit_rank,
    COUNT(visitid) AS numberofvisits,
    EXTRACT(year from (DATE_TRUNC('month', date))) AS year, 
    EXTRACT(month from (DATE_TRUNC('month', date))) AS month
FROM all_sessions
GROUP BY year, month
ORDER BY year, numberofvisits DESC;
```

![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/f47f414c-123f-4624-9573-96d4efb86f11)


## Question 3: 

What is the average time on spent on site for visitors with transactions vs without
```
SELECT ROUND(AVG(a.timeonsite)/60,0) AS avg_time_min
FROM analytics a
JOIN all_sessions al ON al.visitid = a.visitid
WHERE transactions IS NOT NULL
```

![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/49a7b9ea-c710-459d-9b5d-b6b90db61896)# Starting With Data


```
SELECT ROUND(AVG(a.timeonsite)/60,0) AS avg_time_min
FROM analytics a
JOIN all_sessions al ON al.visitid = a.visitid
WHERE transactions IS NULL
```
![image](https://github.com/KelKi3/Final-Project-SQL/assets/104335144/94ac05e4-8d49-4e95-8cdc-120885547def)
