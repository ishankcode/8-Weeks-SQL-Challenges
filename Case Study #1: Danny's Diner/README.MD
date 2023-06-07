# 🍜 Case Study #1: Danny's Diner 

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/21957b47-5563-49c1-ba9b-e78c4dd555c3)

[Visit This link for complete Project Explanantion](https://8weeksqlchallenge.com/case-study-1/)

### Problem Statement
Using Data analyze key information regarding Customers visiting patterns, how much money they’ve spent and also which menu items are their favourite. Using these insights Danny will be able to craft a more personalized menu for his loyal customers. Using these insights will also him decide whether he should expand the existing customer loyalty program or not.

### Entity Relationship Diagrams
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/ca29e2e4-b6b0-475f-8ce6-398e83289c39)

### Challenges

**Q1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT S.customer_id, SUM(price) As "Total_spend $"
FROM sales S INNER JOIN menu M ON S.product_id = M.product_id  
GROUP BY customer_id;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/7e349135-8149-4d6c-8d6e-e6f4e118dcc8)

**Q2 How many days has each customer visited the restaurant?**

````sql
SELECT customer_id, COUNT(DISTINCT order_date) AS "No._of_visits"
FROM sales
GROUP BY customer_id;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/f681c8b1-30eb-4871-bf89-4ecae12627da)

**Q3 What was the first item from the menu purchased by each customer?**

````sql
WITH CTE AS 
(
	SELECT S.customer_id,M.product_name, DENSE_RANK() OVER(PARTITION BY S.customer_id ORDER BY S.order_date) AS rnk
	FROM sales S INNER JOIN menu M ON S.product_id = M.product_id 
)
SELECT DISTINCT customer_id, product_name
FROM CTE
WHERE rnk = 1;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/b45114cc-5ed3-4130-9e2f-8be3842d4593)

**Q4 What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT M.product_id, M.product_name, COUNT(S.order_date) AS Total_purchase
FROM sales S INNER JOIN menu M ON S.product_id = M.product_id 
GROUP BY M.product_id, M.product_name
ORDER BY 3 DESC
LIMIT 1;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/80f74b79-e406-4646-afcf-b46f4b966d21)

**Q5 Which item was the most popular for each customer?**

````sql
WITH CTE AS
(
	SELECT S.customer_id, M.product_id, M.product_name, COUNT(S.order_date) AS Total_purchase,
		RANK() OVER(PARTITION BY S.customer_id ORDER BY COUNT(S.order_date) DESC) AS rnk
	FROM sales S INNER JOIN menu M ON S.product_id = M.product_id 
	GROUP BY S.customer_id,  M.product_id, M.product_name
)
SELECT customer_id, product_name, total_purchase
FROM CTE
WHERE rnk=1;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/1e1d405b-c9df-455b-a21b-5371a75baf67)

**Q6 Which item was purchased first by the customer after they became a member?**

````sql
-- BASICALLY NEED TO FIND THEIR 2ND ORDER (NOT NECESSARILY AS IT MAY BE THE CASE THAT THEY BECOME MEMBER LATER ON)
WITH CTE AS
(
	SELECT S.customer_id, S.order_date, M.product_name
	FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id INNER JOIN
		members ME ON ME.customer_id = S.customer_id
	WHERE S.order_date >= ME.join_date
),

CTE2 AS
(
	SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as rnk
    FROM CTE
)
SELECT *
FROM CTE2
WHERE rnk = 1;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/aed3d851-a8d0-4d3e-bd57-6844470bf40e)

**Q7 Which item was purchased just before the customer became a member?**

````sql
WITH CTE AS
(
	SELECT S.customer_id, S.order_date, M.product_name
	FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id INNER JOIN
		members ME ON ME.customer_id = S.customer_id
	WHERE S.order_date < ME.join_date
),

CTE2 AS
(
	SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY order_date ASC) as rnk
    FROM CTE
)
SELECT customer_id,order_date, product_name
FROM CTE2
WHERE rnk = 1;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/3dee4ac0-2184-43e5-bdf2-4a41c55b7a3a)

**Q8 What is the total items and amount spent for each member before they became a member?**

````sql
SELECT S.customer_id, COUNT(DISTINCT M.product_name) AS unique_menu_items, SUM(M.price) AS total_spent
FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id INNER JOIN
members ME ON ME.customer_id = S.customer_id
WHERE S.order_date < ME.join_date
GROUP BY S.customer_id;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/444821b1-13ad-4eb7-8bf6-33055850bcd9)

**Q9 If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

````sql
SELECT S.customer_id, SUM(CASE WHEN M.product_name = "sushi" THEN 20*M.price ELSE 10*M.price END) AS points
FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id 
GROUP BY S.customer_id
ORDER BY 2 DESC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/7af94173-068b-4f7d-902e-cc0c23abc044)

**Q10 In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not 
just sushi - how many points do customer A and B have at the end of January?**

````sql
SELECT S.customer_id, SUM(CASE WHEN M.product_name = "sushi" THEN 20*M.price ELSE 10*M.price END) AS points
FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id 
GROUP BY S.customer_id
ORDER BY 2 DESC;

-- breaking the problem down
-- calculate all points before join_date
-- first calculate points for all customers for first week of joining
-- calculated points for all customers after 1st week and jan 2021
-- then combine these three and filter only for the specefic customers
WITH CTE AS 
(
	SELECT S.customer_id, S.order_date, M.product_name, (CASE WHEN M.product_name = "sushi" THEN 20*M.price ELSE 10*M.price END) AS points,
		ME.join_date AS promotion_start, date_add(ME.join_date, INTERVAL 1 week) AS promotion_period_end
	FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id INNER JOIN members ME ON S.customer_id = ME.customer_id
	ORDER BY 2 DESC
),
	CTE2 AS 
(
	SELECT *, CASE WHEN order_date BETWEEN promotion_start AND promotion_period_end and product_name != "sushi" THEN 2*points ELSE points END AS final_points
	FROM CTE
)

SELECT customer_id, SUM(final_points) AS Points
FROM CTE2
GROUP BY 1
ORDER BY 2 DESC;

SELECT S.customer_id, SUM(CASE WHEN M.product_name = "sushi" THEN 20*M.price ELSE 10*M.price END) AS points
FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id 
GROUP BY S.customer_id
ORDER BY 2 DESC;

WITH CTE AS 
(
	SELECT S.customer_id, S.order_date, M.product_name, M.price*10 AS points,
		ME.join_date AS promotion_start, date_add(ME.join_date, INTERVAL 1 week) AS promotion_period_end
	FROM Sales S INNER JOIN menu M ON S.product_id = M.product_id INNER JOIN members ME ON S.customer_id = ME.customer_id
	ORDER BY 2 DESC
),
	CTE2 AS 
(
	SELECT *, CASE WHEN order_date BETWEEN promotion_start AND promotion_period_end OR product_name = "sushi" THEN 2*points ELSE points END AS final_points
	FROM CTE
    -- WHERE order_date <= "2021-01-31"
)
SELECT customer_id, SUM(final_points) AS Points
FROM CTE2
GROUP BY 1
ORDER BY 2 DESC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/ef9a3ac0-73d4-4972-9292-1ab7afa379ae)











