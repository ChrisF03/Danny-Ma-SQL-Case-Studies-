## 3. Before and After Analysis

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time. 

Taking the week_date value of *2020-06-15* as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for *2020-06-15* as the start of the period **after** the change and the previous week_date values would be **before**

Using this analysis approach - answer the following questions:

**Filter** <br>
First we need to find the week number corresponding to the given week date of June 15, 2020 so we can use it as a filter.
```sql
    SELECT DISTINCT week_number
    FROM clean_weekly_sales
    WHERE week_date = '2020-06-15'
    AND calendar_year = '2020';
```
| week_number |
| ----------- |
| 25          |

---
**Query #1** <br>
What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
```sql
-- This CTE will provide us with the total sales for each of the 4 weeks before and
-- each of the 4 weeks after the change in packaging:
    WITH sales AS (
    SELECT week_date,
    	   week_number,
           SUM(sales) as total_sales
    FROM clean_weekly_sales
    WHERE week_number BETWEEN 21 AND 28
    AND calendar_year = '2020'
    GROUP BY week_date, week_number
    ORDER BY week_date, week_number
    ),
-- With this CTE we calculate the total sales for all of the 4 weeks before and
-- the 4 weeks after the change in packaging: 
    before_and_after AS (
    SELECT SUM(CASE 
               	WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS before,
      	   SUM(CASE
               	WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) as after
    FROM sales
    )
```
```sql
    SELECT before,
    	   after,
           (after - before) as difference,
    	   ROUND((100 * (after-before) / before), 2) as variance
    FROM before_and_after;
```
| before     | after      | difference | variance |
| ---------- | ---------- | ---------- | -------- |
| 2345878357 | 2318994169 | -26884188  | -1.15    |

---
**Query #2** <br>
What about the entire 12 weeks before and after?
```sql
    WITH sales AS (
    SELECT week_date,
    	   week_number,
           SUM(sales) as total_sales
    FROM clean_weekly_sales
    WHERE week_number BETWEEN 13 AND 37
    AND calendar_year = '2020'
    GROUP BY week_date, week_number
    ORDER BY week_date, week_number
    ),
    before_and_after AS (
    SELECT SUM(CASE 
               	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
      	   SUM(CASE
               	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
    FROM sales
    )
```
```sql
    SELECT before,
    	   after,
           (after - before) as difference,
    	   ROUND((100 * (after-before) / before), 2) as variance
    FROM before_and_after;
```
| before     | after      | difference | variance |
| ---------- | ---------- | ---------- | -------- |
| 7126273147 | 6973947753 | -152325394 | -2.14    |

---
**Query #3** <br>
How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
```sql
    WITH sales AS (
    SELECT calendar_year,
    	   week_number,
           SUM(sales) as total_sales
    FROM clean_weekly_sales
    WHERE week_number BETWEEN 13 AND 37
    GROUP BY calendar_year, week_number
    ORDER BY calendar_year, week_number
    ),
    before_and_after AS (
    SELECT calendar_year,
      	   SUM(CASE 
               	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
      	   SUM(CASE
               	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
    FROM sales
    GROUP BY calendar_year
    ORDER BY calendar_year
    )
```
```sql
    SELECT calendar_year, 
    	   before,
           after,
           (after - before) as difference,
    	   ROUND((100 * (after-before) / before), 2) as variance
    FROM before_and_after
    GROUP BY calendar_year, after , before;
```
| calendar_year | before     | after      | difference | variance |
| ------------- | ---------- | ---------- | ---------- | -------- |
| 2018          | 6396562317 | 6500818510 | 104256193  | 1.63     |
| 2019          | 6883386397 | 6862646103 | -20740294  | -0.30    |
| 2020          | 7126273147 | 6973947753 | -152325394 | -2.14    |
