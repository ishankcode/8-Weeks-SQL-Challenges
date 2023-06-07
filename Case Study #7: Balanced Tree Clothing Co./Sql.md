#  Case Study #7: Balanced Tree Clothing Co.

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/115eaec7-c497-441e-9c90-6507bf8e0832)

[Visit This link for complete Project Explanantion](https://8weeksqlchallenge.com/case-study-7/)

### Problem Statement
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!
Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.
Total 4 Datasets available: product_details, sales, product_hierarchy, product_prices

----


### A. High Level Sales Analysis 

**Q1 What was the total quantity sold for all products?**
````sql
SELECT product_name, SUM(qty) AS total_qty
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1
ORDER BY 2 DESC;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/4782f123-ac27-407d-87d1-067bd37cf1ef)


**Q2 What is the total generated revenue for all products before discounts?**
````sql
SELECT product_name, SUM(S.qty*P.price) AS total_revenue
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1
ORDER BY 2 DESC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/80dbf269-46ff-4998-84cf-b42eaecec673)

**Q3 What was the total discount amount for all products?**
````sql
SELECT product_name, SUM(S.qty*(1-discount/100)*S.price) AS total_revenue
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1
ORDER BY 2 DESC;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/9d44e60d-363e-4714-a17d-7f4930357890)


### B. Transaction Analysis
````sql
Q1 How many unique transactions were there?
SELECT COUNT(DISTINCT txn_id) 
FROM sales;
````

Output: 2500

**Q2 What is the average unique products purchased in each transaction?**
````sql
WITH CTE AS
(
	SELECT txn_id, COUNT(DISTINCT prod_id) AS count_prod
	FROM sales
	GROUP BY 1
)
SELECT round(AVG(count_prod))
FROM CTE;
````

Output: 6

**Q3 What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

   
**Q4 What is the average discount value per transaction?**
````sql
WITH CTE AS 
(
  SELECT txn_id, SUM(price * qty * discount/100) AS total_discount
  FROM sales
  GROUP BY 1
)
SELECT ROUND(AVG(total_discount),2) AS avg_unique_products
FROM CTE;
````

Output(Avg_Unique_Products): 62.49

**Q5 What is the percentage split of all transactions for members vs non-members?**
````sql
WITH CTE AS
(
SELECT member, COUNT(DISTINCT txn_id) as transactions
FROM sales
GROUP BY 1
),
	T AS
(SELECT COUNT(DISTINCT txn_id) AS total FROM sales)
SELECT member, transactions, ROUND(100*transactions/total)
FROM CTE,T;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/4432e6a6-ecbb-43aa-9b94-3ebec15136c0)


**Q6 What is the average revenue for member transactions and non-member transactions?**
````sql
WITH CTE AS
(
SELECT member, txn_id,  SUM(price*qty) as rev
FROM sales
GROUP BY 1,2
)

SELECT member, AVG(rev)
FROM CTE
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/96803e5a-3820-4e62-9789-e03c1e1de171)


### Part C. Product Analysis

**Q1 What are the top 3 products by total revenue before discount?**
````sql
SELECT product_id, product_name, SUM(P.price*qty) AS rev
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 3;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/3f058fab-4a6a-471a-874b-57e3544a4926)


**Q2 What is the total quantity, revenue and discount for each segment?**
````sql
SELECT segment_id, segment_name, SUM(qty) AS total_qty, SUM(P.price*qty) AS total_revenue, SUM(P.price*qty*discount/100) AS disc 
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/1889bb14-6e0b-45df-a593-e55806a7049b)


**Q3 What is the top selling product for each segment?**
````sql
WITH CTE AS 
(SELECT segment_id, segment_name, prod_id, product_name, SUM(qty) AS product_quantity, 
	   DENSE_RANK() OVER(PARTITION BY segment_id, segment_name ORDER BY SUM(qty) DESC) AS rnk
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
group by 1,2,3,4
)

SELECT segment_id, segment_name, prod_id, product_name, product_quantity
FROM CTE
WHERE rnk=1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/0cda424d-6d81-47e2-b374-37888989d110)


**Q4 What is the total quantity, revenue and discount for each category?**
````sql
SELECT category_id, category_name, SUM(qty) AS total_quantity, SUM(P.price*qty) AS revenue, SUM(P.price*discount*qty/100) AS discount
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
group by 1,2;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/5e1372f7-194d-46cd-829e-a2bc27f08cd2)


**Q5 What is the top selling product for each category?**
````sql
WITH CTE AS 
(SELECT category_id, category_name, prod_id, product_name, SUM(qty) AS product_quantity, 
	   DENSE_RANK() OVER(PARTITION BY category_id, category_name ORDER BY SUM(qty) DESC) AS rnk
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
group by 1,2,3,4
)

SELECT category_id, category_name, prod_id, product_name, product_quantity
FROM CTE
WHERE rnk=1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/b229f89b-428b-4284-a4f0-7857ab55b96f)


**Q6 What is the percentage split of revenue by product for each segment?**
````sql
WITH CTE AS
(
SELECT segment_id, segment_name, product_id,product_name, SUM(P.price*qty) AS revenue
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2,3,4
)
SELECT *, ROUND(100*revenue/(SUM(revenue) OVER(PARTITION BY segment_id, segment_name )),2) AS segment_perc
FROM CTE
order by segment_id, segment_perc DESC;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/7d76d8c1-5c94-4938-be18-f494aa98b840)


**Q7 What is the percentage split of revenue by segment for each category?**
````sql
WITH CTE AS
(
SELECT category_id, category_name, segment_id,segment_name, SUM(P.price*qty) AS revenue
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2,3,4
)
SELECT *, ROUND(100*revenue/(SUM(revenue) OVER(PARTITION BY category_id, category_name )),2) AS category_perc
FROM CTE
order by category_id, category_perc DESC;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/e102c9b3-869e-406e-9de3-35849a9650f2)


**Q8 What is the percentage split of total revenue by category?**
````sql
WITH CTE AS
(
SELECT category_id, category_name, SUM(P.price*qty) AS revenue
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2
)
SELECT *, ROUND(100*revenue/(SUM(revenue) OVER()),2) AS cat_perc
FROM CTE
order by category_id, cat_perc DESC;
````
![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/58a37038-304e-49d5-9a0b-4c12c18e94a0)


**Q9 What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity
    of a product was purchased divided by total number of transactions)**
````sql 
WITH CTE AS 
(
SELECT product_id, product_name, COUNT(DISTINCT txn_id) AS prod_txn
FROM product_details P INNER JOIN sales S ON P.product_id = S.prod_id
GROUP BY 1,2
)
SELECT product_id,product_name,prod_txn, ROUND(100*prod_txn/(SELECT COUNT(DISTINCT txn_id) FROM sales),2) AS penetration
FROM CTE;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/d0f0881c-b47c-4bdb-9148-6296d9351a3b)


**Q10 What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**


