#  Case Study #8: Fresh Segments 

![image](https://github.com/ishankcode/8-Weeks-SQL-Challenges/assets/66678343/1481b327-e57e-4a75-a5b2-47307c751858)

[Visit This link for complete Project Explanantion](https://8weeksqlchallenge.com/case-study-8/)

### Problem Statement
Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.
Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.
In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.
Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests. <br />

  2 Datases available: interest_metrics, interest_maps
  
----

## Part A. Data Exploration and Cleansing ------------------------------------*/
**Q1 Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month**
````sql
UPDATE fresh_segments.interest_metrics
SET month_year = STR_TO_DATE(CONCAT('01-', month_year), '%d-%m-%Y');
````

**Q2 What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order
  (earliest to latest) with the null values appearing first?**
````sql
SELECT month_year, COUNT(*)
FROM interest_metrics
GROUP BY 1
ORDER BY 1;
````

**Q3 What do you think we should do with these null values in the fresh_segments.interest_metrics**
-- As there are significant null values that exist, we need to analyze what they might refer to

**Q4 How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table?    What about the other way around?**
````sql
WITH CTE AS 
(
SELECT COUNT(DISTINCT map.id) AS interest_map_only
FROM fresh_segments.interest_map AS map
LEFT OUTER JOIN fresh_segments.interest_metrics AS im ON map.id = im.interest_id
WHERE im.interest_id IS NULL
),
	CTE2 AS 
(
SELECT COUNT(DISTINCT map.id) AS interest_metrics_only
FROM interest_metrics AS im
LEFT OUTER JOIN interest_map AS map ON map.id = im.interest_id
WHERE id IS NULL
)
SELECT *
FROM CTE, CTE2;
````

**Q5 Summarise the id values in the fresh_segments.interest_map by its total record count in this table**
````sql
WITH cte_id_records AS (
SELECT
  id,
  COUNT(*) AS record_count
FROM fresh_segments.interest_map
GROUP BY id
)
SELECT
  id,
  COUNT(record_count) AS id_count
FROM cte_id_records
GROUP BY id
ORDER BY id;
````

**Q6 What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246
#    in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map 
#    except from the id column.**
````sql
-- from analysois done in Qn 4 we observed that all records exist in interet_map, so we can use either LEFT or INNER JOIN.
WITH cte_join AS (
SELECT
  interest_metrics.*,
  interest_map.interest_name,
  interest_map.interest_summary,
  interest_map.created_at,
  interest_map.last_modified
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
)
SELECT * FROM cte_join
WHERE interest_id = 21246;
````

**Q7 Are there any records in your joined table where the month_year value is before the created_at value from the 
   fresh_segments.interest_map table? Do you think these values are valid and why?**
````sql
WITH cte_join AS (
SELECT
  interest_metrics.*,
  interest_map.interest_name,
  interest_map.interest_summary,
  interest_map.created_at,
  interest_map.last_modified
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
  ON interest_metrics.interest_id = interest_map.id
)
SELECT COUNT(*) FROM cte_join
WHERE month_year < created_at;
````

### PART B : Interest Analysis

**Q1 Which interests have been present in all month_year dates in our dataset?**
````sql
-- same id is being repeated in different mont_year, need to calculate the total number of id's which are present in 
-- same number of month_year columns

WITH cte_interest_months AS (
SELECT
  interest_id,
  COUNT(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
)
SELECT
  total_months,
  COUNT(DISTINCT interest_id) AS interest_count
FROM cte_interest_months
GROUP BY total_months
ORDER BY total_months DESC;
````

**Q2 Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which
   total_months value passes the 90% cumulative percentage value?**
````sql
WITH cte_total_months AS 
(
	SELECT interest_id, COUNT(DISTINCT month_year) AS total_months
	FROM interest_metrics
	GROUP BY interest_id
),
	cte_cumalative_perc AS 
(
	SELECT total_months, count(interest_id) AS n_ids,
		-- by using the OVER clause, we can nest aggregate functions.
		round(100 * sum(count(*)) OVER (ORDER BY total_months desc) / sum(count(*)) over(), 2) AS cumalative_perc
	FROM cte_total_months
	GROUP BY total_months
	ORDER BY total_months DESC
)
-- Select results that are >= 90%
SELECT total_months, n_ids, cumalative_perc
FROM cte_cumalative_perc
WHERE cumalative_perc >=90; 
````

**Q3 If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many
   total data points would we be removing?**
````sql
-- So basically need to remove the id's existing with less than 6 months of data
-- IMPORTANT THING I was missing that multiple data points can exist for same uinterest_id
WITH COUNT_CTE AS 
(
	SELECT interest_id, COUNT(DISTINCT month_year) AS total_month
	FROM interest_metrics
	GROUP BY 1
	HAVING total_month < 6
)
SELECT count(interest_id) rows_removed
FROM interest_metrics
WHERE exists( SELECT interest_id
			  FROM COUNT_CTE
			  WHERE COUNT_CTE.interest_id = fresh_segments.interest_metrics.interest_id
			);
````

**Q4 Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months 
   present to a removed interest example for your arguments - think about what it means to have less months present from a segment
   perspective.**

less than 6 months data is basically maybe because of 2 reason, 
     - 1. customer recently joined
     - 2. inconistent data

- if it's the first case then it make sense to exclude the data as we can implememt better strsategies focusing on retenting customers
   for a long time, or learn from these customers about their continued interest and based on our learining implement strategies for attracting new customers
- If it's second case then its bit more tricky to answer and will depend on the business case

**Q5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?**
````sql
SELECT
  month_year,
  COUNT(DISTINCT interest_id) AS unique_ids
FROM fresh_segments.interest_metrics
WHERE month_year IS NOT NULL
GROUP BY 1
ORDER BY 1;
-- Slightly confused in what is actua;ly being asked in problem (Another thing maybe being asked is - same query but not including removed_rows
-- Or asking more detailed answer such as for each month calculating the frquency of each interesr_id
````


### Part C. Segment Analysis 

**Q1 Using the FILTERED dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any month_year?**
````sql
#Only use the maximum composition value for each interest but you must keep the corresponding month_year
DROP TEMPORARY TABLE IF EXISTS filtered_table  ;
CREATE TEMPORARY TABLE filtered_table AS
(
	WITH COUNT_CTE AS 
	(
		SELECT interest_id, COUNT(DISTINCT month_year) AS total_month
		FROM interest_metrics
		GROUP BY 1
		HAVING total_month < 6
	)
SELECT *
FROM interest_metrics
WHERE exists( SELECT interest_id
			  FROM COUNT_CTE
			  WHERE COUNT_CTE.interest_id = fresh_segments.interest_metrics.interest_id
			)
);
WITH TOP_composition AS 
(
	SELECT month_year, interest_id, IM.interest_name, composition, RANK() OVER(ORDER BY composition DESC) AS rnk
	FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
),

	BOTTOM_composition AS
(
	SELECT month_year, interest_id, IM.interest_name, composition, ROW_NUMBER() OVER(ORDER BY composition) AS rnk
	FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
)

SELECT * FROM TOP_composition
WHERE rnk<=10
UNION
SELECT * FROM BOTTOM_composition
WHERE rnk<=10;
````

**Q2 Which 5 interests had the lowest average ranking value?**
````sql
SELECT IM.interest_name, interest_id, AVG(ranking), COUNT(*)
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
GROUP BY 1,2
ORDER BY 3 ASC
LIMIT 5;
````

**Q3 Which 5 interests had the largest standard deviation in their percentile_ranking value?**
````sql
SELECT IM.interest_name, interest_id, STDDEV(percentile_ranking), COUNT(*)
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;
````

**Q4 For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and
   its corresponding year_month value? Can you describe what is happening for these 5 interests?**
````sql
WITH CTE_FILTERED AS
(
	SELECT IM.interest_name, interest_id, STDDEV(percentile_ranking), COUNT(*)
	FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
	GROUP BY 1,2
	ORDER BY 3 DESC
	LIMIT 5
)

SELECT interest_id, MIN(percentile_ranking), MAX(percentile_ranking)
FROM interest_metrics
WHERE interest_id IN (SELECT interest_id FROM CTE_FILTERED) 
GROUP BY 1;
````
-- FROM tnis we observed that the five customers identified with highest standard deviation in percentile ranking
-- has a very high range. Thus this can be used to verify our previous results 


**Q5 How would you describe our customers in this segment based off their composition and ranking values? What sort of products or 
   services should we show to these customers and what should we avoid?**
   
- open ended question, In broader sense only looking at the top/bottom id's are sort of similar to looking at outliers. 
- based on business objectve can become very important for the analysis. 
- So based on the anlysis as from part 2 - we can see that generally interest involving outdoor activities have the lowest rankings which
  indicates these have the most composition compared to other interests. And this can also be seen from out analysis in part 1 
  as top 5 activitoes by interest are all related to work then travelling or gym

### Part D. Index Analysis

#The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.
# Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

**Q1 What is the top 10 interests by the average composition for each month?**
````sql
WITH CTE AS 
(SELECT month_year, interest_id, IM.interest_name, composition/ROUND(index_value,2) AS avg_composition,
	   DENSE_RANK() OVER(PARTITION BY month_year ORDER BY composition/ROUND(index_value,2) DESC) AS rnk
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
WHERE month_year IS NOT NULL
)

SELECT *
FROM CTE
WHERE rnk<=10;
````

**Q2 For all of these top 10 interests - which interest appears the most often?**
````sql
WITH CTE AS 
(SELECT month_year, interest_id, IM.interest_name, composition/ROUND(index_value,2) AS avg_composition,
	   DENSE_RANK() OVER(PARTITION BY month_year ORDER BY composition/ROUND(index_value,2) DESC) AS rnk
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
WHERE month_year IS NOT NULL
),
	CTE_F AS
(
	SELECT *
	FROM CTE
	WHERE rnk<=10
),
	appearance AS
(
	SELECT interest_name, interest_id, COUNT(*)	AS appearances
	FROM CTE_F
	GROUP BY 1,2 
	ORDER BY 3 DESC
)
SELECT *
from appearance
WHERE appearances = (SELECT MAX(appearances) FROM appearance);
````

**Q3 What is the average of the average composition for the top 10 interests for each month?**
````sql
WITH CTE AS 
(SELECT month_year, interest_id, IM.interest_name, composition/ROUND(index_value,2) AS avg_composition,
	   DENSE_RANK() OVER(PARTITION BY month_year ORDER BY composition/ROUND(index_value,2) DESC) AS rnk
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
WHERE month_year IS NOT NULL
),
	CTE_F AS 
(	
	SELECT *	
	FROM CTE
	WHERE rnk<=10
)
SELECT month_year, ROUND(AVG(avg_composition),2) AS avg_avg_composition
FROM CTE_F
GROUP BY 1;
````

**Q4 What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the 
   previous top ranking interests in the same output shown below.**
````sql
-- NEED TO SHOW IN output THE PREVIOUS MONTHS AVG_COMPOSITON AS WELL
WITH CTE AS 
(
SELECT month_year, interest_name, ROUND(composition / index_value,2) AS index_composition,
		 RANK() OVER (PARTITION BY month_year ORDER BY composition/index_value DESC) AS index_rank
FROM interest_metrics I INNER JOIN interest_map IM ON I.interest_id = IM.id
WHERE month_year IS NOT NULL
),

	CTE_F AS 
(
SELECT month_year, interest_name, index_composition, 
	   AVG(index_composition) OVER (ORDER BY month_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_avg_3months,
       LAG(interest_name) OVER (ORDER BY month_year) AS int_1_month_ago,
       LAG(index_composition) OVER (ORDER BY month_year) AS composition_1_month_ago,
       LAG(interest_name,1) OVER (ORDER BY month_year) AS int_2_month_ago,
       LAG(index_composition,1) OVER (ORDER BY month_year) AS composition_2_month_ago
FROM CTE
WHERE index_rank = 1 
)

SELECT month_year, interest_name, index_composition, rolling_avg_3months, 
	   CONCAT(int_1_month_ago, ' : ', composition_1_month_ago) AS 1_month_ago,
	   CONCAT(int_2_month_ago, ' : ', composition_2_month_ago) AS 2_month_ago
FROM CTE_F
WHERE month_year >= "2018-09-01";
````

**Q5 Provide a possible reason why the max average composition might change from month to month? Could it signal something is not 
   quite right with the overall business model for Fresh Segments?**
-  There can be multiple factors regarding this:
-  1. Seasonal - Based on season peoples interest gets changes to enjoy that particular season

