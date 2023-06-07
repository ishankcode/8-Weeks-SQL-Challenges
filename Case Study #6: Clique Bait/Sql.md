#  Case Study #6: Clique Bait

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/c76e76f3-82ab-4d67-82b2-7aa58d83d551)

[Visit This link for complete Project Explanantion](https://8weeksqlchallenge.com/case-study-6/)

### Problem Statement
Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry! In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

----

### Part B - Digital Analysis

**Q1 How many users are there?**
````sql
SELECT COUNT(DISTINCT user_id) 
FROM users;
````
OUTPUT: 500

**Q2 How many cookies does each user have on average?**
````sql
WITH CTE AS 
(
SELECT user_id, COUNT(cookie_id) AS cookie_count 
FROM users 
GROUP BY 1
)
SELECT SUM(cookie_count)/COUNT(*) AS avg_cookies
FROM CTE;
````

Output (Avg_Cookies): 3.5640

**Q3 What is the unique number of visits by all users per month?**
````sql
SELECT MONTH(event_time), COUNT(DISTINCT visit_id)
FROM events
GROUP BY 1 ;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/0a239360-4aba-447b-afb6-b0781d818a91)


**Q4 What is the number of events for each event type?**
````sql
SELECT event_type, COUNT(*) 
FROM events 
GROUP BY 1
ORDER BY 2 desc;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/8678462f-ed34-4c53-98b2-4d96d8335f25)


**Q5 What is the percentage of visits which have a purchase event?**
````sql
WITH CTE AS
(
	SELECT COUNT(DISTINCT visit_id) AS pur_visit
	FROM events
    WHERE event_type = 3
)
SELECT ROUND(100*pur_visit/(SELECT COUNT(DISTINCT visit_id) FROM events),2) as PURCHASE_Perc
FROM CTE;
````

OUTPUT(Purchase_Perc): 49.86

**Q6 What is the percentage of visits which view the checkout page but do not have a purchase event?**
````sql
WITH cte_visits_with_checkout_and_purchase_flags AS (
  SELECT
    visit_id,
    -- confirm that the only events that occur on the checkout page are views
    MAX(CASE WHEN event_type = 1 AND page_id = 11 THEN 1 ELSE 0 END) AS checkout_flag,
    MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase_flag
  FROM clique_bait.events
  GROUP BY visit_id
)
SELECT
  ROUND(100 * SUM(CASE WHEN purchase_flag = 0 THEN 1 ELSE 0 END)/ COUNT(*), 2) AS checkout_without_purchase_percentage
FROM cte_visits_with_checkout_and_purchase_flags
WHERE checkout_flag = 1;
````

Output(checkout_without_purchase_percentage): 27.55

**Q7 What are the top 3 pages by number of views?**
````sql
SELECT E.page_id, COUNT(DIstinct visit_id)
FROM events E INNER JOIN page_hierarchy P ON E.page_id = P.page_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/38c7d0f6-1c8b-4a6e-b2ab-d2f53cd206ca)


**Q8 What is the number of views and cart adds for each product category?**
````sql
SELECT product_category, 
	SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS count_views,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS count_addcart
FROM page_hierarchy P INNER JOIN events E ON P.page_id = E.page_id
WHERE product_category IS NOT NULL
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/585ce30c-a173-437f-b6bf-6218a7475a7b)


**Q9 What are the top 3 products by purchases?**
````sql
-- Here we dont have records of the product names with the purchase event.
-- instead we need to see if visit_id has associate purchase event or not and check if these items have been added to the cart
-- if so then basically these items have been purchased. (So join 3 tables events, page_hierarchy)
SELECT product_id, page_name AS product_name, COUNT(E.event_type) AS num_purchases
FROM page_hierarchy P JOIN events E ON P.page_id = E.page_id 
WHERE E.event_type = '2' AND visit_id IN (SELECT DISTINCT visit_id FROM events WHERE event_type = 3) AND product_id >0
GROUP BY 1,2
order by 3 DESC
LIMIT 3;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/32776142-4ec8-42c3-a250-f02bf66e0e0c)


### PRODUCT FUNNEL ANALYSIS 

**Using a single SQL query - create a new output table which has the following details:
    - #1. How many times was each product viewed?
    - #2. How many times was each product added to cart?
    - #3. How many times was each product added to a cart but not purchased (abandoned)?
    - #4. How many times was each product purchased?
    - #5. Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual           products.**
````sql
DROP TABLE IF EXISTS product_info;
DROP TABLE IF EXISTS product_catgeory_info;

CREATE TEMPORARY TABLE product_info 
WITH page_events AS
(
SELECT visit_id, product_id, page_name AS product, product_category,
	   SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_view,
       SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_add
FROM events E INNER JOIN page_hierarchy P ON E.page_id = P.page_id
WHERE product_id IS NOT NULL
GROUP BY 1,2,3, 4
),

	purchase_event AS
(
	SELECT DISTINCT visit_id AS visit_id
    FROM events
    WHERE event_type = 3
),

	combined AS
(
	SELECT PE.visit_id, product_id, product, product_category, page_view, cart_add,
			CASE WHEN P.visit_id IS NOT NULL THEN 1 ELSE 0 END AS purchase
    FROM page_events PE LEFT JOIN  purchase_event P ON PE.visit_id = P.visit_id
)


SELECT product, product_category, SUM(page_view) AS views, SUM(CASE WHEN cart_add IN (1,2) then 1 else 0 end) AS carts,			
		SUM(CASE WHEN cart_add IN (1,2)  and purchase = 0 THEN 1 ELSE 0 END) AS abondoned,
		SUM(CASE WHEN cart_add IN (1,2) AND purchase = 1 THEN 1 ELSE 0 END) AS purchased
FROM combined
GROUP BY 1,2;
''''

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/dc1d054a-87f5-430b-8a1c-eadae23aec62)


````sql
# second table
CREATE TEMPORARY TABLE product_category_info
SELECT product_category, SUM(views) AS views, SUM(carts) AS carts, SUM(abondoned) AS abondoned, SUM(purchased) AS purchased
FROM product_info
GROUP BY 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/9e42b8dd-7902-4963-9ca0-91aa9e04d444)


**Q1 Which product had the most views, cart adds and purchases?**
````sql
-- cannot reference temporary table multiple times in same query, can replace it with cte 
WITH page_events AS
(
SELECT visit_id, product_id, page_name AS product, product_category,
	   SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_view,
       SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_add
FROM events E INNER JOIN page_hierarchy P ON E.page_id = P.page_id
WHERE product_id IS NOT NULL
GROUP BY 1,2,3, 4
),

	purchase_event AS
(
	SELECT DISTINCT visit_id AS visit_id
    FROM events
    WHERE event_type = 3
),

	combined AS
(
	SELECT PE.visit_id, product_id, product, product_category, page_view, cart_add,
			CASE WHEN P.visit_id IS NOT NULL THEN 1 ELSE 0 END AS purchase
    FROM page_events PE LEFT JOIN  purchase_event P ON PE.visit_id = P.visit_id
), 
	final_cal AS
(
	SELECT product, product_category, SUM(page_view) AS views, SUM(CASE WHEN cart_add IN (1,2) then 1 else 0 end) AS carts,			
			SUM(CASE WHEN cart_add IN (1,2)  and purchase = 0 THEN 1 ELSE 0 END) AS abondoned,
			SUM(CASE WHEN cart_add IN (1,2) AND purchase = 1 THEN 1 ELSE 0 END) AS purchased
	FROM combined
	GROUP BY 1,2
)
SELECT *
FROM 
	(SELECT product AS most_views FROM final_cal ORDER BY views DESC LIMIT 1) AS T,
    (SELECT product AS most_added FROM final_cal ORDER BY carts DESC LIMIT 1) AS T2,
    (SELECT product AS most_purchased FROM final_cal ORDER BY purchased DESC LIMIT 1) AS T3;
 
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/d2b38a29-d253-4c83-a21d-3162a8d92b44)


**Q2 Which product was most likely to be abandoned?**
````sql
-- NEED TO CALCULATE THE LIKELYHOOD probability - abondened/cart_added
SELECT product, abondoned/carts AS abondonded_likelyhood
FROM product_info
ORDER BY 2 DESC
limit 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/d929b8da-bb50-4436-a87b-0b7b58da0212)


**Q3 Which product had the highest view to purchase percentage?**
````sql
SELECT product, purchased/views AS view_purchase_perc
FROM product_info
ORDER BY 2 DESC
limit 1;
````

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/41e35a4b-c294-488c-a48d-30c91480018c)


**Q4 What is the average conversion rate from view to cart add?**
````sql
SELECT AVG(carts/views) AS views_to_add_conversion
FROM product_info;
````

Output(views_to_add_conversion): 0.5317019

**Q5 What is the average conversion rate from cart add to purchase?**
````sql
SELECT AVG(purchased/carts) AS carts_to_purchased_conversion
FROM product_info;
````

Output(carts_to_purchased_conversion): 0.75927977

### PART D - CAMPAIGN ANALYSIS 

**Generate a table that has 1 single row for every unique visit_id record and has the following columns:
    - 1. user_id
    - 2. visit_id
    - 3. visit_start_time: the earliest event_time for each visit
    - 4. page_views: count of page views for each visit
    - 5. cart_adds: count of product cart add events for each visit
    - 6. purchase: 1/0 flag if a purchase event exists for each visit
    - 7. campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
    - 8. impression: count of ad impressions for each visit
    - 9. click: count of ad clicks for each visit
    - 10. Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were
          added to the cart (hint: use the sequence_number)**
          
````sql
DROP TABLE IF EXISTS campaign_analysis_table;
CREATE TEMPORARY TABLE campaign_analysis_table
SELECT
  users.user_id,
  events.visit_id,
  MIN(events.event_time) AS visit_start_time,
  SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
  MAX(CASE WHEN events.event_type = 3 THEN 1 ELSE 0 END) AS purchase,
  campaign_identifier.campaign_name,
  MAX(CASE WHEN events.event_type = 4 THEN 1 ELSE 0 END) AS impression,
  MAX(CASE WHEN events.event_type = 5 THEN 1 ELSE 0 END) AS click,
 GROUP_CONCAT(
    CASE
      WHEN page_hierarchy.product_id IS NOT NULL AND event_type = 2
        THEN page_hierarchy.page_name
      ELSE NULL END,
    ', ' ORDER BY events.sequence_number
  ) AS cart_products
FROM clique_bait.events
INNER JOIN clique_bait.users
  ON events.cookie_id = users.cookie_id
LEFT JOIN clique_bait.campaign_identifier
  ON events.event_time BETWEEN campaign_identifier.start_date AND campaign_identifier.end_date
LEFT JOIN clique_bait.page_hierarchy
  ON events.page_id = page_hierarchy.page_id
GROUP BY
  users.user_id,
  events.visit_id,
  campaign_identifier.campaign_name
  ORDER BY user_id;
  ````
  
  ![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/2b34f63f-f6b5-4943-a15d-892564dc2698)

  
----

