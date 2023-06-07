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


