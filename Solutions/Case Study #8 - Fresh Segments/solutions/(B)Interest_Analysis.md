## B. Interest Analysis

**1) Which interests have been present in all month_year dates in our dataset?**
```sql
WITH interest AS (
SELECT interest_id, 
  	   COUNT(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
GROUP BY interest_id
)
SELECT total_months,
  	   COUNT(DISTINCT interest_id) as total_interests
FROM interest
GROUP BY total_months
ORDER BY total_interests DESC
LIMIT 1;
```
| total_months | total_interests |
| ------------ | --------------- |
| 14           | 480             |

---

**2) Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?**
```sql
WITH interest_month AS (
SELECT interest_id,
  	   COUNT(month_year) as total_months
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
),
interest_count AS (
SELECT total_months,
       COUNT(DISTINCT interest_id) AS interest_count
FROM interest_month
GROUP BY total_months
),
cumulative AS (
SELECT total_months,
  	   interest_count,
  	   ROUND(100 * SUM(interest_count) OVER (ORDER BY total_months DESC) / (SUM(interest_count) OVER ()),2) AS cumulative_percentage
FROM interest_count
)
SELECT *
FROM cumulative
WHERE cumulative_percentage >= 90;
```
| total_months | interest_count | cumulative_percentage |
| ------------ | -------------- | --------------------- |
| 6            | 33             | 90.85                 |
| 5            | 38             | 94.01                 |
| 4            | 32             | 96.67                 |
| 3            | 15             | 97.92                 |
| 2            | 12             | 98.92                 |
| 1            | 13             | 100.00                |

---

**3) If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?**
```sql
WITH interest_count AS (
SELECT interest_id,
       COUNT(month_year) AS month_year_total
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
HAVING COUNT(month_year) < 7
)
SELECT COUNT(*) rows_removed
FROM fresh_segments.interest_metrics
WHERE EXISTS (SELECT interest_id 
              FROM interest_count
              WHERE interest_count.interest_id = fresh_segments.interest_metrics.interest_id);
```
| rows_removed |
| ------------ |
| 598          |

---

**4) Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.**

- I would say it makes sense to remove the data points since are relatively small amounts and would allow us to better focus on the more successful interests. The rows we are removing dont exhibit strong or constant interest from users.

```sql
WITH cte AS (
SELECT month_year, 
	   COUNT(interest_id) AS number_of_interests
FROM fresh_segments.interest_metrics
WHERE month_year IS NOT NULL
AND interest_id::INTEGER IN (SELECT interest_id::INTEGER
                             FROM fresh_segments.interest_metrics
                             GROUP BY interest_id 
                             HAVING COUNT(interest_id) > 6)
GROUP BY month_year
ORDER BY month_year
),
cte2 AS (
SELECT t1.month_year,
	   COUNT(t2.interest_id) AS before_excl,
       t1.number_of_interests AS after_excl,
       (COUNT(t2.interest_id) - t1.number_of_interests) as amount_excl
FROM cte as t1
JOIN fresh_segments.interest_metrics as t2 ON
t1.month_year = t2.month_year
WHERE t1.month_year IS NOT NULL
GROUP BY t1.month_year, t1.number_of_interests
ORDER BY t1.month_year
)
SELECT month_year,
	   before_excl,
       after_excl,
       amount_excl,
       ROUND(100 * (amount_excl::NUMERIC / before_excl::NUMERIC), 2) as percent_excl
FROM cte2
GROUP BY month_year, before_excl, after_excl, amount_excl;
```
| month_year | before_excl | after_excl | amount_excl | percent_excl |
| ---------- | ----------- | ---------- | ----------- | ------------ |
| 01-2019    | 973         | 957        | 16          | 1.64         |
| 02-2019    | 1121        | 1050       | 71          | 6.33         |
| 03-2019    | 1136        | 1046       | 90          | 7.92         |
| 04-2019    | 1099        | 1013       | 86          | 7.83         |
| 05-2019    | 857         | 817        | 40          | 4.67         |
| 06-2019    | 824         | 792        | 32          | 3.88         |
| 07-2018    | 729         | 701        | 28          | 3.84         |
| 07-2019    | 864         | 823        | 41          | 4.75         |
| 08-2018    | 767         | 743        | 24          | 3.13         |
| 08-2019    | 1149        | 1033       | 116         | 10.10        |
| 09-2018    | 780         | 767        | 13          | 1.67         |
| 10-2018    | 857         | 844        | 13          | 1.52         |
| 11-2018    | 928         | 919        | 9           | 0.97         |
| 12-2018    | 995         | 976        | 19          | 1.91         |

---

**5) After removing these interests - how many unique interests are there for each month?**
```sql
WITH cte AS (
SELECT month_year, 
	   COUNT(interest_id) AS number_of_interests
FROM fresh_segments.interest_metrics
WHERE month_year IS NOT NULL
AND interest_id::INTEGER IN (SELECT interest_id::INTEGER
                             FROM fresh_segments.interest_metrics
                             GROUP BY interest_id 
                             HAVING COUNT(interest_id) > 6)
GROUP BY month_year
ORDER BY month_year
)
SELECT t1.month_year,
	   COUNT(t2.interest_id) AS before_excl,
       t1.number_of_interests AS after_excl
FROM cte as t1
JOIN fresh_segments.interest_metrics as t2 ON
t1.month_year = t2.month_year
WHERE t1.month_year IS NOT NULL
GROUP BY t1.month_year, t1.number_of_interests
ORDER BY t1.month_year
```
| month_year | before_excl | after_excl |
| ---------- | ----------- | ---------- |
| 01-2019    | 973         | 957        |
| 02-2019    | 1121        | 1050       |
| 03-2019    | 1136        | 1046       |
| 04-2019    | 1099        | 1013       |
| 05-2019    | 857         | 817        |
| 06-2019    | 824         | 792        |
| 07-2018    | 729         | 701        |
| 07-2019    | 864         | 823        |
| 08-2018    | 767         | 743        |
| 08-2019    | 1149        | 1033       |
| 09-2018    | 780         | 767        |
| 10-2018    | 857         | 844        |
| 11-2018    | 928         | 919        |
| 12-2018    | 995         | 976        |
