


/*---------------------- Part A data cleaninsing Steps -----------------------------*/
DROP TABLE IF EXISTS clean_weekly_sales;
CREATE TABLE clean_weekly_sales AS
(
	WITH CTE AS 
	(
		SELECT *, STR_TO_DATE(week_date, '%d/%m/%Y') AS week_date_f
		FROM weekly_sales
	)
	SELECT week_date_f, WEEK(week_date_f) AS week_number, MONTH(week_date_f) AS month_number, YEAR(week_date_f) AS calendar_year,
			region, platform, 
            CASE WHEN segment IS NULL THEN 'unknown' ELSE segment END AS segment,
            CASE WHEN RIGHT(segment,1) = '1' THEN 'Young_adults' 
				 WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
                 WHEN RIGHT(segment,1) IN ('3','4') THEN 'Retirees'
                 ELSE 'Unknown'
			END AS age_band,
			CASE WHEN LEFT(segment,1) = 'C' THEN 'Couples'
				 WHEN LEFT(segment,1) = 'F' THEN 'Families'
                 ELSE 'Unknown'
			END AS demographic,
            customer_type,transactions, sales,
            ROUND(sales/transactions,2) AS avg_transaction
    FROM CTE
);


/* ------------------------- Part B Data Exploration -------------------------------*/
#Q1 What day of the week is used for each week_date value?
SELECT DISTINCT DAYNAME(week_date_f)
FROM clean_weekly_sales;
-- MONDAY

#Q2 What range of week numbers are missing from the dataset?
SELECT DISTINCT week_number
FROM clean_weekly_sales
ORDER BY 1;
-- Now <=12 days dont exost and more than 35 don't exist
-- to get actual list of numbers create a temp table with values 1 - 52 and do not in operation

#Q3 How many total transactions were there for each year in the dataset?
SELECT calendar_year, SUM(transactions)
FROM clean_weekly_sales
GROUP BY 1
ORDER BY 1;

#Q4 What is the total sales for each region for each month?
SELECT region, month_number, SUM(sales)
FROM clean_weekly_sales
GROUP BY 2,1
ORDER BY 2,1;

#Q5 What is the total count of transactions for each platform
SELECT platform, SUM(transactions)
FROM clean_weekly_sales
GROUP BY 1;

#Q6 What is the percentage of sales for Retail vs Shopify for each month?
WITH CTE_t AS 
(
	SELECT CONCAT(month_number, '-', calendar_year) AS month_year, SUM(sales) AS total_sales, SUM(CASE WHEN platform = 'retail' THEN sales ELSE 0 END) AS retail_sales,
    SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END) AS shoppify_sales
    FROM clean_weekly_sales
    GROUP BY 1
    ORDER BY 1
)

SELECT month_year, ROUND(100*(retail_sales/total_sales),2) AS retail, ROUND(100*(shoppify_sales/total_sales),2) AS shoppify
FROM CTE_t;

#Q7 What is the amount and percentage of sales by demographic for each year in the dataset?
SELECT calendar_year, demographic, SUM(sales) AS yearly_sales,
	   ROUND(100*(SUM(sales)/(SUM(SUM(sales)) OVER(PARTITION BY calendar_year))),2) AS perc_sales
FROM clean_weekly_sales
GROUP BY 1,2 
ORDER BY 1,2;

#Q8 Which age_band and demographic values contribute the most to Retail sales?
-- age_band
SELECT age_band, SUM(sales),ROUND(100*(SUM(sales)/(SUM(SUM(sales)) OVER())),2) AS ageband_sales 
FROM clean_weekly_sales
GROUP BY 1;

-- demographic band
SELECT demographic, SUM(sales),ROUND(100*(SUM(sales)/(SUM(SUM(sales)) OVER())),2) AS ageband_sales 
FROM clean_weekly_sales
GROUP BY 1;

-- both
SELECT age_band, demographic, SUM(sales),ROUND(100*(SUM(sales)/(SUM(SUM(sales)) OVER())),2) AS ageband_sales 
FROM clean_weekly_sales
GROUP BY 1,2 ;

#Qn 9  Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
-- No we cannot do that
SELECT calendar_year, platform, ROUND(SUM(sales)/SUM(transactions),2)
FROM clean_weekly_sales
GROUP BY 1,2;


/*--------------------------------- Part c ANALYSIS --------------------------------------*/
#Q1 What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in 
 #actual values and percentage of sales?
WITH CTE1 AS
(
	SELECT CASE WHEN week_number BETWEEN 21 AND 24 then '1_before'
				WHEN week_number BETWEEN 26 AND 29 THEN '2_after'
                ELSE 'at_week_25'
			END AS period,
		SUM(sales) AS total_sales
	FROM clean_weekly_sales
    WHERE week_number BETWEEN 21 AND 29 AND calendar_year = '2020'
    GROUP BY 1
),
	CTE2 AS 
(
	SELECT period, total_sales - LAG(total_sales) OVER(ORDER BY period) AS sales_diff,
			ROUND(100*((total_sales/LAG(total_sales) OVER(ORDER BY period) - 1)),2) AS perc
	FROM CTE1
    WHERE period != 'at_week_25'
)

SELECT sales_diff, perc
FROM CTE2
WHERE sales_diff IS NOT NULL;

#Q2 What about the entire 12 weeks before and after?
-- Can use the same code as in Q1

#Q3 How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
WITH cte_4weeks AS (
  SELECT
    calendar_year,
    CASE
      WHEN week_number BETWEEN 21 AND 24 THEN '1.Before'
      WHEN week_number BETWEEN 25 AND 28 THEN '2.After'
      END AS period_name,
    SUM(sales) AS total_sales,
    SUM(transactions) AS total_transactions,
    SUM(sales) / SUM(transactions) AS avg_transaction_size
  FROM data_mart.clean_weekly_sales
  WHERE week_number BETWEEN 21 and 28
  GROUP BY
    calendar_year,
    period_name
),
cte_calculations AS (
  SELECT
    calendar_year,
    period_name,
    total_sales - LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name) AS sales_diff,
    ROUND(
      100 * ((total_sales/ LAG(total_sales) OVER (PARTITION BY calendar_year ORDER BY period_name)) - 1),2) AS sales_change
  FROM cte_4weeks
)
SELECT
  calendar_year,
  sales_diff,
  sales_change
FROM cte_calculations
WHERE sales_change IS NOT NULL AND sales_diff IS NOT NULL
ORDER BY calendar_year;

-- And similarly can be done for 12 weeks period as well
