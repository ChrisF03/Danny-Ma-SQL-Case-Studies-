## 4. Bonus Question

Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

* region
* platform
* age_band
* demographic
* customer_type

### Region
```sql
WITH sales_by_region AS (
SELECT region,
  	   week_number,
       SUM(sales) as total_sales
FROM clean_weekly_sales
WHERE week_number BETWEEN 13 AND 37
AND calendar_year = '2020'
GROUP BY region, week_number
ORDER BY region, week_number
),
before_and_after AS (
SELECT region,
  	   SUM(CASE 
           	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
  	   SUM(CASE
           	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
FROM sales_by_region
GROUP BY region
)
SELECT region,
	   before,
	   after,
       (after - before) as difference,
	   ROUND((100 * (after-before) / before), 2) as variance
FROM before_and_after
GROUP BY region, before, after
ORDER BY variance ASC;
```
| region        | before     | after      | difference | variance |
| ------------- | ---------- | ---------- | ---------- | -------- |
| ASIA          | 1637244466 | 1583807621 | -53436845  | -3.26    |
| OCEANIA       | 2354116790 | 2282795690 | -71321100  | -3.03    |
| SOUTH AMERICA | 213036207  | 208452033  | -4584174   | -2.15    |
| CANADA        | 426438454  | 418264441  | -8174013   | -1.92    |
| USA           | 677013558  | 666198715  | -10814843  | -1.60    |
| AFRICA        | 1709537105 | 1700390294 | -9146811   | -0.54    |
| EUROPE        | 108886567  | 114038959  | 5152392    | 4.73     |

---

### Platform
```sql
WITH sales_by_platform AS (
SELECT platform,
  	   week_number,
       SUM(sales) as total_sales
FROM clean_weekly_sales
WHERE week_number BETWEEN 13 AND 37
AND calendar_year = '2020'
GROUP BY platform, week_number
ORDER BY platform, week_number
),
before_and_after AS (
SELECT platform,
  	   SUM(CASE 
           	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
  	   SUM(CASE
           	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
FROM sales_by_platform
GROUP BY platform
)
SELECT platform,
	   before,
	   after,
       (after - before) as difference,
	   ROUND((100 * (after-before) / before), 2) as variance
FROM before_and_after
GROUP BY platform, before, after
ORDER BY variance ASC;
```
| platform | before     | after      | difference | variance |
| -------- | ---------- | ---------- | ---------- | -------- |
| Retail   | 6906861113 | 6738777279 | -168083834 | -2.43    |
| Shopify  | 219412034  | 235170474  | 15758440   | 7.18     |

---

### Age Band
```sql
WITH sales_by_age_band AS (
SELECT age_band,
  	   week_number,
       SUM(sales) as total_sales
FROM clean_weekly_sales
WHERE week_number BETWEEN 13 AND 37
AND calendar_year = '2020'
GROUP BY age_band, week_number
ORDER BY age_band, week_number
),
before_and_after AS (
SELECT age_band,
  	   SUM(CASE 
           	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
  	   SUM(CASE
           	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
FROM sales_by_age_band
GROUP BY age_band
)
SELECT age_band,
	   before,
	   after,
       (after - before) as difference,
	   ROUND((100 * (after-before) / before), 2) as variance
FROM before_and_after
GROUP BY age_band, before, after
ORDER BY variance ASC;
```
| age_band     | before     | after      | difference | variance |
| ------------ | ---------- | ---------- | ---------- | -------- |
| unknown      | 2764354464 | 2671961443 | -92393021  | -3.34    |
| Middle Aged  | 1164847640 | 1141853348 | -22994292  | -1.97    |
| Retirees     | 2395264515 | 2365714994 | -29549521  | -1.23    |
| Young Adults | 801806528  | 794417968  | -7388560   | -0.92    |

---

### Demographic
```sql
WITH sales_by_demographic AS (
SELECT demographic,
  	   week_number,
       SUM(sales) as total_sales
FROM clean_weekly_sales
WHERE week_number BETWEEN 13 AND 37
AND calendar_year = '2020'
GROUP BY demographic, week_number
ORDER BY demographic, week_number
),
before_and_after AS (
SELECT demographic,
  	   SUM(CASE 
           	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
  	   SUM(CASE
           	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
FROM sales_by_demographic
GROUP BY demographic
)
SELECT demographic,
	   before,
	   after,
       (after - before) as difference,
	   ROUND((100 * (after-before) / before), 2) as variance
FROM before_and_after
GROUP BY demographic, before, after
ORDER BY variance ASC;
```
| demographic | before     | after      | difference | variance |
| ----------- | ---------- | ---------- | ---------- | -------- |
| unknown     | 2764354464 | 2671961443 | -92393021  | -3.34    |
| Families    | 2328329040 | 2286009025 | -42320015  | -1.82    |
| Couples     | 2033589643 | 2015977285 | -17612358  | -0.87    |

---

### Customer Type
```sql
WITH sales_by_customer_type AS (
SELECT customer_type,
  	   week_number,
       SUM(sales) as total_sales
FROM clean_weekly_sales
WHERE week_number BETWEEN 13 AND 37
AND calendar_year = '2020'
GROUP BY customer_type, week_number
ORDER BY customer_type, week_number
),
before_and_after AS (
SELECT customer_type,
  	   SUM(CASE 
           	WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before,
  	   SUM(CASE
           	WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) as after
FROM sales_by_customer_type
GROUP BY customer_type
)
SELECT customer_type,
	   before,
	   after,
       (after - before) as difference,
	   ROUND((100 * (after-before) / before), 2) as variance
FROM before_and_after
GROUP BY customer_type, before, after
ORDER BY variance ASC;
```
| customer_type | before     | after      | difference | variance |
| ------------- | ---------- | ---------- | ---------- | -------- |
| Guest         | 2573436301 | 2496233635 | -77202666  | -3.00    |
| Existing      | 3690116427 | 3606243454 | -83872973  | -2.27    |
| New           | 862720419  | 871470664  | 8750245    | 1.01     |
