![image](https://github.com/iqraburki/sql-foodie-fi-case-study/assets/169553712/7aa00206-3ad9-40a4-8d79-e6363cf924fc)# sql-foodie-fi-case-study
---Query 1: How many customers has Foodie-Fi ever had?

SELECT COUNT(DISTINCT customer_id)
AS total_customers
FROM foodiefie.subscriptions;

---result:
total_customers	
1000	

----Query 2 What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.

SELECT EXTRACT(MONTH from start_date) AS months, COUNT(*)
FROM foodiefie.subscriptions 
where plan_id = 0 
Group by months
order by months;

---Result
months	COUNT(*)
1	      88
2	      68
3	      94
4     	81
5	      88
6	      79
7     	89
8     	88
9     	87
10    	79
11    	75
12	    84

----	March (3) has the highest number of trial plans, while February (2) has the lowest number of trial plans.


--- QUERY 3: What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

SELECT plan_id, count(*)
as events_2021 
from subscriptions
 where EXTRACT(YEAR FROM start_date) >= '2021-01-01'
GROUP BY plan_id
 ORDER BY plan_id

 ---Result

 plan_id	events_2021
1        	8
2	        60
3	        63
4	        71

---- There is no trial plan recorded for 2021.

--- QUERY 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

Select count(*) as churn_count , 
ROUND (count(*) / 10, 1) AS churn_percentage 
from subscriptions
 where plan_id = 4

---Result
churn_count  	churn_percentage
307	          30.7

--- 	307 customers, or 30.7% of the total customers have churned from Foodie-fi.

---- Query 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

WITH CTE AS (
SELECT 
customer_id,
plan_name,
ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date ASC) as rn
FROM subscriptions as S
INNER JOIN plans as P on S.plan_id = P.plan_id
)
SELECT 
COUNT(DISTINCT customer_id) 
as customers_churned_afer_trial,
ROUND((COUNT(DISTINCT customer_id) / (SELECT COUNT(DISTINCT customer_id)
FROM subscriptions))*100,0) as churn_after_trial_percent
FROM CTE
WHERE rn = 2
AND plan_name = 'churn';

---- Result
customers_churned_afer_trial	churn_after_trial_percent
92	                           9

--- 92 customers, or 9% of the total customers, have churned straight after their initial trial  from Foodie-fi.


