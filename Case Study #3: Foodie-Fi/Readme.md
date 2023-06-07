# 🥑 Case Study #3: Foodie-Fi

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/1d8cb5dd-7def-4458-beb4-363ec2b87249)





/* ------------------------------------------------ Customer Journey --------------------------------------------------*/
#Q1 Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each
# customer’s onboarding journey.
SELECT P.plan_id, plan_name, price, customer_id, start_date
FROM plans P INNER JOIN subscriptions S ON P.plan_id = S.plan_id
order by 4;

/* -----------------------------------------------------Data Analysis Questions--------------------------------------------------*/
# Q1 How many customers has Foodie-Fi ever had?
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions;

#Q2 What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?
SELECT MIN(start_date), COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions
WHERE plan_id = 0
GROUP BY MONTH(start_date);

#Q3 What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
SELECT plan_id,
       plan_name,
       count(*) AS 'count of events'
FROM subscriptions
JOIN plans USING (plan_id)
WHERE year(start_date) > 2020
GROUP BY 1,2
ORDER BY 3 ASC ;

#Q4 What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
SELECT SUM(CASE WHEN P.plan_id = 4 THEN 1 ELSE 0 END) AS count_customers,
		ROUND(100*SUM(CASE WHEN P.plan_id = 4 THEN 1 ELSE 0 END)/COUNT(DISTINCT customer_id),1) AS perc_customers
FROM plans P INNER JOIN subscriptions S ON P.plan_id = S.plan_id;

#HQ5 ow many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
WITH next_plan_cte AS
(
	SELECT *, lead(plan_id, 1) over(PARTITION BY customer_id ORDER BY start_date) AS next_plan
	FROM subscriptions
),
     churners AS
(	
	SELECT *
	FROM next_plan_cte
	WHERE next_plan=4 AND plan_id=0
)
SELECT count(customer_id) AS 'churn after trial count', round(100 *count(customer_id)/
						(SELECT count(DISTINCT customer_id) AS 'distinct customers' FROM subscriptions), 2) AS 'churn percentage'
FROM churners;

#Q6 What is the number and percentage of customer plans after their initial free trial?
WITH ranked_plans AS 
(
	SELECT customer_id,plan_id,ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date ASC) AS plan_rank
	FROM foodie_fi.subscriptions
)
SELECT plans.plan_id, plans.plan_name, COUNT(*) AS customer_count,
	ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER ()) AS percentage
FROM ranked_plans
INNER JOIN foodie_fi.plans
  ON ranked_plans.plan_id = plans.plan_id
WHERE plan_rank = 2
GROUP BY plans.plan_id, plans.plan_name
ORDER BY plans.plan_id;

#Q7 What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
-- Basiclly need to identify the customer status based on the current plan
WITH latest_update AS
(
	SELECT *, row_number() over(PARTITION BY customer_id ORDER BY start_date DESC) AS latest_plan
   FROM subscriptions
   JOIN plans USING (plan_id)
   WHERE start_date <='2020-12-31' 
)
SELECT plan_id, plan_name, count(customer_id) AS customer_count,
       round(100*count(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions), 2) AS percentage_breakdown
FROM latest_update
WHERE latest_plan = 1
GROUP BY 1,2 
ORDER BY plan_id;

#Q8 How many customers have upgraded to an annual plan in 2020?
SELECT COUNT(DISTINCT customer_id)
FROM subscriptions
WHERE YEAR(start_date)= 2020 AND plan_id = 3;
-- here there can be multiple options to this question that plan was upgraded after the customer was started at plan 0 or at plan1
-- in that case can use row_number() to keep track of customers updates as done previously

#Q9 How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
WITH CTE1 AS
( 
	SELECT customer_id, MIN(start_date) AS customer_start
	FROM subscriptions
	GROUP BY 1
),
	CTE2 AS
(
		SELECT customer_id, start_date AS plan_3_start
        FROM subscriptions
        WHERE plan_id = 3
)
SELECT ROUND(AVG(DATEDIFF(plan_3_start,customer_start)))
FROM CTE1 C1 INNER JOIN CTE2 C2 ON C1.customer_id = C2.customer_id;


#Q10 Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
WITH annual_plan AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
),
trial AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
),
annual_days AS (
SELECT A.customer_id, DATEDIFF(A.start_date, T.start_date) AS duration
FROM annual_plan A
INNER JOIN trial T
  ON A.customer_id = T.customer_id
)
SELECT CASE WHEN duration < 30 THEN "0 - 30 Days"
			WHEN duration BETWEEN 30 AND 60 THEN "30 - 60 Days"
            WHEN duration BETWEEN 60 AND 90 THEN "60 - 90 Days"
            WHEN duration BETWEEN 90 AND 120 THEN "90 - 120 Days"
            WHEN duration BETWEEN 120 AND 150 THEN "120 - 150 Days"
            WHEN duration BETWEEN 150 AND 180 THEN "150 - 180 Days"
            WHEN duration BETWEEN 180 AND 210 THEN "180 - 210 Days"
            WHEN duration BETWEEN 210 AND 240 THEN "210 - 240 Days"
            WHEN duration BETWEEN 240 AND 270 THEN "240 - 270 Days"
            WHEN duration BETWEEN 270 AND 300 THEN "270 - 300 Days"
            WHEN duration BETWEEN 300 AND 330 THEN "300 - 330 Days"
            WHEN duration BETWEEN 330 AND 360 THEN "330 - 360 Days"
		END AS break_period,
        COUNT(customer_id)
FROM annual_days
GROUP BY 1;

#Q11 How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
WITH CTE AS 
(	
	SELECT customer_id, plan_id, start_date, LAG(plan_id) OVER(PARTITION BY customer_id ORDER BY start_date) AS prev_plan
	FROM subscriptions
)
SELECT COUNT(*) AS downgraded_count
FROM CTE 
where plan_id = 1 AND prev_plan = 2 AND YEAR(start_date) = 2020;


#How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
