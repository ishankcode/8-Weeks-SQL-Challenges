-----

**Q1 If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has 
 Pizza Runner made so far if there are no delivery fees?**
````sql
#Not including thoese orders which were cancelled by the resteraunt
SELECT SUM(CASE WHEN pizza_id = 1 THEN 10 ELSE 12 END) AS earned_amt
FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
WHERE (cancellation IS NULL OR cancellation ="NaN" OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation'))	;
````
Output (Earned_amount): 126

**Q2 What if there was an additional $1 charge for any pizza extras? example Add cheese is $1 extra**
  ````sql
  SELECT SUM(CASE WHEN extras = 'NaN' OR extras IS NULL THEN 0 ELSE LENGTH(REGEXP_REPLACE(extras, '[^0-9]', '')) END) + 
		SUM(CASE WHEN pizza_id = 1 THEN 10 ELSE 12 END) AS earned_amt
FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
WHERE (cancellation IS NULL OR cancellation ="NaN" OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation'))	;
````
Output (Earned_amount): 130

**Q3  If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per
 kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**
````sql
WITH CTE1 AS 
(	SELECT SUM(CASE WHEN pizza_id = 1 THEN 10 ELSE 12 END) AS earned_amt
	FROM customers_orders C INNER JOIN runner_orders R ON C.order_id = R.order_id
	WHERE (cancellation IS NULL OR cancellation ="NaN" OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation'))
),
	CTE2 AS
(
	SELECT SUM(runner_payment) AS runner_payment
	FROM 
    (
		SELECT MAX(distance)*0.3 AS runner_payment
		FROM runner_orders
		WHERE distance is NOT Null
		GROUP BY order_id
	) T
)
SELECT earned_amt - runner_payment AS leftover_revenue
FROM CTE1, CTE2;
````

Output (Leftover_revenue): 82.44

