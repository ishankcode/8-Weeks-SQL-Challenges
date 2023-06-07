----

**Q1 What are the standard ingredients for each pizza?

````sql
WITH CTE AS 
(
	SELECT pizza_id, REGEXP_SUBSTR(toppings, '[^,]+', 1, n) AS topping
	FROM pizza_recipes
	INNER JOIN (
		SELECT 1 AS n UNION ALL
		SELECT 2 AS n UNION ALL
		SELECT 3 AS n UNION ALL
		SELECT 4 AS n UNION ALL
		SELECT 5 AS n UNION ALL
		SELECT 6 AS n UNION ALL
		SELECT 7 AS n UNION ALL
		SELECT 8 AS n
	) AS nums ON n <= CHAR_LENGTH(toppings) - CHAR_LENGTH(REPLACE(toppings, ',', '')) + 1
	ORDER BY pizza_id, topping
),

	CTE2 AS 
(
	SELECT pizza_id,topping_name
	FROM CTE C INNER JOIN pizza_toppings P ON C.topping = P.topping_id
)

SELECT pizza_id, GROUP_CONCAT(topping_name ORDER BY topping_name ASC SEPARATOR ', ') AS toppings_recipes
FROM CTE2
GROUP BY 1
ORDER BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/903d6b9b-d656-484f-9621-f03eac7483a4)


**Q2 What was the most commonly added extra?**
````sql
-- first again need to convert csv to list and then count the most common extras
WITH CTE AS 
(
	SELECT order_id, REGEXP_SUBSTR(extras, '[^,]+', 1, n) AS extras_list
	FROM customers_orders
	INNER JOIN (
		SELECT 1 AS n UNION ALL
		SELECT 2 AS n UNION ALL
		SELECT 3 AS n UNION ALL
		SELECT 4 AS n UNION ALL
		SELECT 5 AS n UNION ALL
		SELECT 6 AS n UNION ALL
		SELECT 7 AS n UNION ALL
		SELECT 8 AS n
	) AS nums ON n <= CHAR_LENGTH(extras) - CHAR_LENGTH(REPLACE(extras, ',', '')) + 1
	ORDER BY order_id, extras
),

	CTE2 AS 
(
	SELECT order_id, C.extras_list, P.topping_id, P.topping_name
	FROM CTE C INNER JOIN pizza_toppings P ON C.extras_list = P.topping_id
)

SELECT topping_name, COUNT(order_id) AS extras_ordered
FROM CTE2
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/48afa99a-637d-413d-b9bf-d1fe65d899cf)


**Q3 What was the most common exclusion?**
````sql
-- Same process as above
WITH CTE AS 
(
	SELECT order_id, REGEXP_SUBSTR(exclusions, '[^,]+', 1, n) AS exclusions_list
	FROM customers_orders
	INNER JOIN (
		SELECT 1 AS n UNION ALL
		SELECT 2 AS n UNION ALL
		SELECT 3 AS n UNION ALL
		SELECT 4 AS n UNION ALL
		SELECT 5 AS n UNION ALL
		SELECT 6 AS n UNION ALL
		SELECT 7 AS n UNION ALL
		SELECT 8 AS n
	) AS nums ON n <= CHAR_LENGTH(exclusions) - CHAR_LENGTH(REPLACE(exclusions, ',', '')) + 1
	ORDER BY order_id, exclusions_list
),

	CTE2 AS 
(
	SELECT order_id, C.exclusions_list, P.topping_id, P.topping_name
	FROM CTE C INNER JOIN pizza_toppings P ON C.exclusions_list = P.topping_id
)

SELECT topping_name, COUNT(order_id) AS exclusions_ordered
FROM CTE2
GROUP BY 1
ORDER BY 2 DESC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/13d76b48-2394-4dd0-b196-38c7057124c9)

----

