----

**#Q1 How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```sql
SELECT EXTRACT(WEEK FROM registration_date) AS registration_week,
COUNT(*) AS runners
FROM pizza_runner.runners
GROUP BY 1
ORDER BY 1;
````

**Q12 What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

````sql
WITH CTE AS 
(
	SELECT DISTINCT order_id,MAX(customer_id),order_time
    FROM customers_orders
    GROUP BY 1,3    
),

	CTE2 AS 
(	
	SELECT C.order_id, pickup_time, order_time,
		(CASE WHEN HOUR(pickup_time)=0 THEN 24*60 ELSE HOUR(pickup_time)*60 END + MINUTE(pickup_time) + SECOND(pickup_time)/60) - 
		(CASE WHEN HOUR(order_time)=0 THEN 24*60 ELSE HOUR(order_time)*60 END + MINUTE(order_time)+SECOND(order_time)/60) AS duration
	FROM CTE C INNER JOIN runner_orders R ON C.order_id = R.order_id
	WHERE pickup_time != 'null'	
)	
SELECT AVG(duration) AS avg_pickup_time
FROM CTE2;
````

**Q3 Is there any relationship between the number of pizzas and how long the order takes to prepare?**
````sql
WITH CTE AS 
(
	SELECT DISTINCT order_id,COUNT(pizza_id) AS num_pizza,order_time
    FROM customers_orders
    GROUP BY 1,3    
),

	CTE2 AS 
(	
	SELECT C.order_id, num_pizza, pickup_time, order_time,
		(CASE WHEN HOUR(pickup_time)=0 THEN 24*60 ELSE HOUR(pickup_time)*60 END + MINUTE(pickup_time) + SECOND(pickup_time)/60) - 
		(CASE WHEN HOUR(order_time)=0 THEN 24*60 ELSE HOUR(order_time)*60 END + MINUTE(order_time)+SECOND(order_time)/60) AS duration
	FROM CTE C INNER JOIN runner_orders R ON C.order_id = R.order_id
	WHERE pickup_time != 'null'	
)	
SELECT order_id, duration, num_pizza
FROM CTE2
ORDER BY 3;
````

-- Althogh there definately seems that more pizzas increase the duration (order_time-pickup_time) but cannot be said for certain as
-- it might be irrelevant and the increase might be because drivers [revious delivery was far way

**Q4 What was the average distance travelled for each customer?**

````sql
SELECT customer_id, ROUND(AVG(distance),1) AS avg_dist
FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
WHERE duration != 'null'
GROUP BY 1;
````

**Q5 What was the difference between the longest and shortest delivery times for all orders?**

````sql
	SELECT (SUBSTR(duration,2,1)) AS dur 
    FROM runner_orders
    WHERE duration != "null";
 ````

**Q6 What was the average speed for each runner for each delivery and do you notice any trend for these values?**
```sql
SELECT distance/duration as dur
FROM runner_orders;
````

**Q7 What is the successful delivery percentage for each runner?**
````sql
WITH CTE1 AS 
(
	SELECT runner_id ,COUNT(*) AS c1
	FROM runner_orders
	WHERE duration !="null"
	GROUP BY 1
),

	CTE2 AS 
(	SELECT runner_id ,COUNT(*) AS c2
	FROM runner_orders
	GROUP BY 1
)

SELECT C1.runner_id, ROUND(c1/c2*100, 2)
FROM CTE1 C1 INNER JOIN CTE2 C2 ON C1.runner_id = C2.runner_id;
````

----


