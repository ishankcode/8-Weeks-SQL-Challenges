#  Case Study #4: Data Bank

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/de5b6211-b7e2-4896-981c-4a661cd27fc8)

[Visit This link for complete Project Explanantion](https://8weeksqlchallenge.com/case-study-4/)

### Problem Statement
Danny created a new streaming service that only had food related content - something like Netflix but with only cooking shows! Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world! This case study focuses on using subscription style digital data to answer important business questions.

### Entity Relationship Diagram
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/6cc9443a-d333-4ae3-84ce-fc0c8dae0af2)

----

### Part A. Customer Nodes Exploration 

**Q1 How many unique nodes are there on the Data Bank system?**
````sql
#SELECT COUNT(DISTINCT node_id) 
#FROM customer_nodes
-- This is wrong because same node id can exist in different regions
-- so basic distinct count won't work

WITH CTE AS 
(
SELECT COUNT(DISTINCT node_id) AS C
FROM customer_nodes
GROUP BY region_id, node_id
)

select sum(C) AS Unique_Nodes
FROM CTE;
````

**Q2 What is the number of nodes per region?**
````sql
SELECT region_name,COUNT(DISTINCT node_id)
FROM customer_nodes C INNER JOIN regions R ON C.region_id = R.region_id 
GROUP BY region_name;
````

**Q3 How many customers are allocated to each region?**
````sql
SELECT region_name,COUNT(DISTINCT customer_id)
FROM customer_nodes C INNER JOIN regions R ON C.region_id = R.region_id 
GROUP BY region_name;
````

**#Q4 How many days on average are customers reallocated to a different node?**


**#Q5 What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

----

### Part B. Customer Transactions 

**Q1 What is the unique count and total amount for each transaction type?**
````sql
SELECT txn_type, SUM(txn_amount) As total_amount, COUNT(*) AS txn_count 
FROM customer_transactions
GROUP BY 1;
````

**Q2 What is the average total historical deposit counts and amounts for all customers?**
````sql
WITH CTE AS 
(
	SELECT customer_id, SUM(txn_amount) AS amounts, COUNT(txn_amount) AS counts
	FROM customer_transactions
	WHERE txn_type = "deposit"
	GROUP BY customer_id
)

SELECT SUM(amounts)/SUM(counts), SUM(counts)/COUNT(*)
FROM CTE;
````

- Should not take averga of average !!!
- Important concept here to calculate the historical avaerage of count - Firstly i calculate total amounts and total transactions
- per customer, and then to calculate average i divided the total sum by the total transactions instead of total number of customers

**Q3 For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
````sql
WITH CTE AS 
(
SELECT MONTH(txn_date) AS months, customer_id, 
	   SUM(CASE WHEN txn_type = "deposit" THEN 1 ELSE 0 END) AS count_dep,
       SUM(CASE WHEN txn_type = "purchase" THEN 1 ELSE 0 END) AS count_pur,
       SUM(CASE WHEN txn_type = "withdrawal" THEN 1 ELSE 0 END) AS count_with
FROM Customer_transactions
GROUP BY 1, 2
ORDER BY 1
)

SELECT months, COUNT(DISTINCT customer_id)
FROM CTE
WHERE count_dep >1 AND (count_pur >=1 OR count_with >=1)
GROUP BY 1;
````

**Q4 What is the closing balance for each customer at the end of the month?**
````sql
WITH CTE1 AS 
(
	SELECT customer_id, MONTH(txn_date) AS months, SUM(CASE WHEN txn_type = "deposit" THEN txn_amount ELSE -1*txn_amount END) AS balance
	FROM customer_transactions
	GROUP BY 1,2
	ORDER BY 1,2
),
	CTE2 AS 
(
	SELECT DISTINCT(customer_id), months
    FROM (SELECT 1 as months UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 ) AS all__months
    CROSS JOIN 
    (SELECT DISTINCT customer_id
    FROM customer_transactions
    ) as ALL_cust
    ORDER BY 1,2
)
SELECT C2.customer_id, C2.months, IFNULL(balance,0) AS balance_contribution,
		SUM(balance) OVER(PARTITION BY customer_id ORDER BY months  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) As ending_bal
FROM CTE2 C2 LEFT JOIN CTE1 C1 ON C2.customer_id = C1.customer_id AND C2.months = C1.months;
````

**Q5 Comparing the closing balance of a customer’s first month and the closing balance from their second nth, what percentage of customers: <br />
     -- Have a negative first month balance? <br />
     -- Have a positive first month balance? <br />
     -- Increase their opening month’s positive closing balance by more than 5% in the following month? <br />
     -- Reduce their opening month’s positive closing balance by more than 5% in the following month?   <br />
     -- Move from a positive balance in the first month to a negative balance in the second month?**
````sql     
WITH CTE1 AS 
(
	SELECT customer_id, MONTH(txn_date) AS months, SUM(CASE WHEN txn_type = "deposit" THEN txn_amount ELSE -1*txn_amount END) AS balance
	FROM customer_transactions
	GROUP BY 1,2
	ORDER BY 1,2
),
	CTE2 AS 
(
	SELECT DISTINCT(customer_id), months
    FROM (SELECT 1 as months UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 ) AS all__months
    CROSS JOIN 
    (SELECT DISTINCT customer_id
    FROM customer_transactions
    ) as ALL_cust
    ORDER BY 1,2
),
	CTE3 AS 
(	
	SELECT C2.customer_id, C2.months, IFNULL(balance,0) AS balance_contribution,
			LAG(balance) OVER(PARTITION BY customer_id ORDER BY months) as prev_balance, 
			SUM(balance) OVER(PARTITION BY customer_id ORDER BY months  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) As ending_bal
	FROM CTE2 C2 LEFT JOIN CTE1 C1 ON C2.customer_id = C1.customer_id AND C2.months = C1.months
)

SELECT ROUND(100*SUM(CASE WHEN months =1 AND ending_bal <0 THEN 1 ELSE 0 END)/COUNT(DISTINCT customer_id),2) AS negative_firsmonth_bal,
	   ROUND(100*SUM(CASE WHEN months =1 AND ending_bal >0 THEN 1 ELSE 0 END)/COUNT(DISTINCT customer_id),2) AS positive_firsmonth_bal,
       ROUND(100*SUM(CASE WHEN months = 2 AND 100*((balance_contribution-prev_balance)/prev_balance) > 5 THEN 1 ELSE 0 END)/COUNT(*),2) AS increase_5_pc,
       ROUND(100*SUM(CASE WHEN months = 2 AND 100*((prev_balance - balance_contribution)/prev_balance) > 5 THEN 1 ELSE 0 END)/COUNT(*),2) AS decrease_5_pc,
       ROUND(100*SUM(CASE WHEN balance_contribution < prev_balance THEN 1 ELSE 0 END)/COUNT(*),2) AS neg_to_pos
FROM CTE3;
````


----
