#### A. Pizza Metrics
-- ALTER TABLE customers_orders MODIFY COLUMN order_time DATETIME;
-- UPDATE customers_orders SET order_time = STR_TO_DATE(order_time, '%m/%d/%Y %H:%i:%s');
-- SELECT order_time, STR_TO_DATE(order_time, '%m/%d/%Y %H:%i:%s')
-- FROM customers_orders

**Q1 How many pizzas were ordered? **

````sql
SELECT COUNT(*) FROM customers_orders;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/02bc05a4-e3ae-41b6-8fb9-77a5939ba9af)


**Q2 How many unique customer orders were made? **

````sql
SELECT COUNT(DISTINCT pizza_id) FROM customers_orders;
````

Output: 2

**Q3 How many successful orders were delivered by each runner? **

````sql
SELECT runner_id, COUNT(*) AS successful_count
FROM runner_orders
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/e857923d-9117-4b4d-88a5-ce50887e9165)


**Q4 How many of each type of pizza was delivered? **

````sql
-- Although there can be cases where orders can be cancelled
SELECT pizza_name, COUNT(*) AS pizza_count
FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
	INNER JOIN pizza_names P ON C.pizza_id = P.pizza_id
WHERE cancellation IS NULL OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/9ccce12c-4d4c-4e42-afd9-75f67843fa94)


**Q5 How many Vegetarian and Meatlovers were ordered by each customer? **

````sql
SELECT customer_id, SUM(CASE WHEN pizza_id =1 THEN 1 ELSE 0 END) AS meat_lovers,
					SUM(CASE WHEN pizza_id =2 THEN 1 ELSE 0 END) AS veggie_lovers
FROM customers_orders
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/e208d433-129c-4800-a4d0-91529e66fb8b)


**Q6 What was the maximum number of pizzas delivered in a single order? **

````sql
WITH CTE AS 
(	SELECT C.order_id, COUNT(*) AS count_pizzas, DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS rnk
	FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
	where cancellation IS NULL OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
	GROUP BY 1
)
SELECT count_pizzas
FROM CTE
WHERE rnk=1;
````
Output: 3

**Q7 For each customer, how many delivered pizzas had at least 1 change and how many had no changes? **

````sql
WITH CTE AS 
(
	SELECT C.customer_id, SUM(CASE WHEN (C.exclusions IS NULL OR C.exclusions = "") AND  (C.extras IS NULL OR C.extras = "" OR 
					C.extras = "NaN") THEN 1 ELSE 0 end ) AS no_change, 
                    COUNT(*) - SUM(CASE WHEN (C.exclusions IS NULL OR C.exclusions = "") AND  (C.extras IS NULL OR C.extras = "" OR 
					C.extras = "NaN") THEN 1 ELSE 0 end ) AS changes
	FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
	where cancellation IS NULL OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
	GROUP BY 1
)
select *
from CTE;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/478c5325-dfe4-4c45-9555-d20857d118db)



**Q8 How many pizzas were delivered that had both exclusions and extras? **

````sql
SELECT COUNT(*)
FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
WHERE (cancellation IS NULL OR cancellation ="NaN" OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation'))		
	AND (C.exclusions IS NOT NULL AND C.exclusions != ""  AND C.extras IS NOT NULL AND C.extras != "" AND C.extras != "NaN");
````
Output: 1


**Q9 What was the total volume of pizzas ordered for each hour of the day? **

````sql
SELECT HOUR(order_time) AS hour_of_datE, COUNT(*) AS pizza_count
FROM customers_orders
GROUP BY 1
ORDER BY 1 ASC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/41c5b1cf-32f7-41b7-93fb-bad92c7cb0e7)

**Q10 What was the volume of orders for each day of the week? **

````sql
SELECT DAYNAME(order_time) , COUNT(*)
FROM customers_orders
GROUP BY 1
ORDER BY 1 DESC;
````


----
