<a href="https://8weeksqlchallenge.com/case-study-2/"> <img align="right" width="300" height="300" src="https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%232%20-%20Pizza%20Runner/2.png"></a>

## Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Case Study
This case study has LOTS of questions - they are broken up by area of focus including:

- [Pizza Metrics](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%232%20-%20Pizza%20Runner/solutions/(A)Pizza_Metrics.md)
- [Runner and Customer Experience](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%232%20-%20Pizza%20Runner/solutions/(B)Runner_and_Customer_Experience.md)
- Ingredient Optimisation
- Pricing and Ratings
- Bonus DML Challenges (DML = Data Manipulation Language)

The Pizza Runner case study focuses on the following SQL skills:

- Common table expressions
- Group by aggregates
- Table joins
- String transformations
- Dealing with null values
- Regular expressions

## Entity Relationship Diagram
![Pizza Runner(entity relationship)](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/assets/103148784/8df3ccec-7e96-4e61-92cf-28882d1ef0e6)
---
<details>
<summary>
Data Cleaning
</summary>
  
## Data Cleaning
### <ins> Customer_Orders </ins>
- The customer_orders table has inconsistent data types.  We must first clean the data before answering any questions. 
- The exclusions and extras columns contain values that are either 'null' (text), NULL (data type) or '' (empty).
- We will create a temporary table where all forms of null will be transformed to NULL (data type).

**Query #1**
  
```sql
    SELECT *
    FROM pizza_runner.customer_orders;
```
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       | null   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       | null   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

---
**Query #2**
```sql
    DROP TABLE IF EXISTS new_customer_orders;
```
```sql
    CREATE TEMP TABLE new_customer_orders AS (
    	SELECT
    		order_id,
    		customer_id,
    		pizza_id,
    	CASE
    		WHEN exclusions = ''
    			OR exclusions LIKE 'null' THEN Null
    		ELSE exclusions
    	END AS exclusions,
    	CASE
    		WHEN extras = ''
    			OR extras LIKE 'null' THEN Null
    		ELSE extras
    	END AS extras,
    		order_time
    FROM
    	pizza_runner.customer_orders
    );
```
```sql
    SELECT * FROM new_customer_orders;
```
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        | NULL       | NULL   | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        | NULL       | NULL   | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        | NULL       | NULL   | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        | NULL       | NULL   | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          | NULL   | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          | NULL   | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          | NULL   | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | NULL       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | NULL       | NULL   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | NULL       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | NULL       | NULL   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | NULL       | NULL   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |
---
---
### <ins> Runner_Orders </ins>
- The runner_order table has inconsistent data types.  We must first clean the data before answering any questions. 
- The distance and duration columns have text and numbers.  We will remove the text values and convert to numeric values.
- We will convert all 'null' (text) and 'NaN' values in the cancellation column to NULL (data type).
- We will convert the pickup_time (varchar) column to a timestamp data type.

**Query #1**
  
```sql
    SELECT * 
    FROM pizza_runner.runner_orders;
```
| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

---
**Query #2**
```sql
    DROP TABLE IF EXISTS new_runner_orders;
```
```sql
    CREATE TEMP TABLE new_runner_orders AS (
    SELECT
    		order_id,
    		runner_id,
    		CASE
    			WHEN pickup_time LIKE 'null' THEN NULL
    		ELSE pickup_time
    	END::timestamp AS pickup_time,
        NULLIF(regexp_replace(distance, '[^0-9.]', '', 'g'), '')::NUMERIC AS distance,
        NULLIF(regexp_replace(duration, '[^0-9.]', '', 'g'), '')::NUMERIC AS duration,
    		CASE
    			WHEN cancellation LIKE 'null'
    				OR cancellation LIKE 'NaN' 
    				OR cancellation LIKE '' THEN NULL
    		ELSE cancellation
    	END AS cancellation
    FROM
    	pizza_runner.runner_orders
    );
```
```sql
    SELECT * FROM new_runner_orders;
```
| order_id | runner_id | pickup_time              | distance | duration | cancellation            |
| -------- | --------- | ------------------------ | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01T18:15:34.000Z | 20       | 32       | NULL                    |
| 2        | 1         | 2020-01-01T19:10:54.000Z | 20       | 27       | NULL                    |
| 3        | 1         | 2020-01-03T00:12:37.000Z | 13.4     | 20       | NULL                    |
| 4        | 2         | 2020-01-04T13:53:03.000Z | 23.4     | 40       | NULL                    |
| 5        | 3         | 2020-01-08T21:10:57.000Z | 10       | 15       | NULL                    |
| 6        | 3         | NULL                     | NULL     | NULL     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08T21:30:45.000Z | 25       | 25       | NULL                    |
| 8        | 2         | 2020-01-10T00:15:02.000Z | 23.4     | 15       | NULL                    |
| 9        | 2         | NULL                     | NULL     | NULL     | Customer Cancellation   |
| 10       | 1         | 2020-01-11T18:50:20.000Z | 10       | 10       | NULL                    |

</details>

---
