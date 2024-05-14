![image](https://github.com/iqraburki/sql-foodie-fi-case-study/assets/169553712/ce0277b8-4db6-4087-a619-ed8e8121ccd4)# sql-foodie-fi-case-study
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

---- Query 6: What is the number and percentage of customer plans after their initial free trial?

WITH CTE AS (
SELECT
customer_id,
plan_name,
ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date ASC) as rn
FROM subscriptions as S
INNER JOIN plans as P on P.plan_id = S.plan_id
)
SELECT 
plan_name,
COUNT(customer_id) as customer_count,
ROUND((COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM CTE))*100,1) as customer_percent
FROM CTE
WHERE rn = 2
GROUP BY plan_name;

--- Result
plan_name   	     customer_count     	customer_percent
basic monthly    	546                	54.6
pro annual       	37	                 3.7
pro monthly      	325                	32.5
churn            	92                  9.2

---	More than 80% of customers are on paid plans.

---- Query 7: What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

WITH CTE AS (
SELECT *
,ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date DESC) as rn
FROM subscriptions
WHERE start_date <= '2020-12-31'
)
SELECT 
plan_name,
COUNT(customer_id) as customer_count,
ROUND((COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM CTE))*100,1) 
as percent_of_customers
FROM CTE
INNER JOIN plans 
as P on CTE.plan_id = P.plan_id
WHERE rn = 1
GROUP BY plan_name;

--- Result 
plan_name	      customer_count	   percent_of_customers
trial	          19	               1.9
basic monthly  	224              	22.4
pro monthly	    326              	32.6
pro annual     	195              	19.5
churn	          236              	23.6

----	More people upgraded to the pro monthly plan, but fewer people signed up for the trial plan.

---- Query 8: How many customers have upgraded to an annual plan in 2020?

SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions 
WHERE plan_id = 3 AND EXTRACT(YEAR from start_date ) = '2020'

--- Result

total_customers
195

---- 	195 customers upgraded to an annual plan in 2020.

--- Query 9: How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

WITH trial_plan AS (
    SELECT customer_id,
           start_date AS trial_date
    FROM subscriptions
    WHERE plan_id = 0),
annual_plan AS (
    SELECT customer_id,
           start_date AS annual_date
    FROM subscriptions
    WHERE plan_id = 3)
SELECT ROUND(AVG(ABS(DATEDIFF(trial_date, annual_date))), 0) AS avg_days_to_upgrade
FROM trial_plan tp
JOIN annual_plan ap ON tp.customer_id = ap.customer_id;

--- Result 
avg_days_to_upgrade
105



