<a href="https://8weeksqlchallenge.com/case-study-2/"> <img align="right" width="300" height="300" src="https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%232%20-%20Pizza%20Runner/2.png"></a>

## Introduction
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## Case Study
This case study has LOTS of questions - they are broken up by area of focus including:

- Pizza Metrics
- Runner and Customer Experience
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

<details>
<summary>
A. Pizza Metrics
</summary>

## A. Pizza Metrics

**Query #1** <br>
How many pizzas were ordered?
```sql
    SELECT COUNT(pizza_id)
    FROM new_customer_orders;
```
| count |
| ----- |
| 14    |

---
**Query #2** <br>
How many unique customer orders were made?
```sql
    SELECT COUNT(DISTINCT order_id)
    FROM new_customer_orders;
```
| count |
| ----- |
| 10    |

---
**Query #3** <br>
How many successful orders were delivered by each runner?
```sql
    SELECT runner_id, 
    	   COUNT(DISTINCT order_id) AS Successful_Orders
    FROM new_runner_orders
    WHERE cancellation IS NULL
    GROUP by runner_id
    ORDER BY runner_id ASC;
```
| runner_id | successful_orders |
| --------- | ----------------- |
| 1         | 4                 |
| 2         | 3                 |
| 3         | 1                 |

---
**Query #4** <br>
How many of each type of pizza was delivered?
```sql
    SELECT pizza_names.pizza_name,
    	   COUNT(new_customer_orders.pizza_id) AS Total
    FROM new_customer_orders
    JOIN pizza_runner.pizza_names ON
    pizza_names.pizza_id = new_customer_orders.pizza_id
    JOIN new_runner_orders ON
    new_runner_orders.order_id = new_customer_orders.order_id
    WHERE cancellation IS NULL
    GROUP BY pizza_names.pizza_name
    ORDER BY Total DESC;
```
| pizza_name | total |
| ---------- | ----- |
| Meatlovers | 9     |
| Vegetarian | 3     |

---
**Query #5** <br>
How many Vegetarian and Meatlovers were ordered by each customer?
```sql
    SELECT customer_id,
    	   SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS Meat_Lovers,
           SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS Vegetarian
    FROM new_customer_orders
    GROUP BY customer_id
    ORDER BY customer_id;
```
| customer_id | meat_lovers | vegetarian |
| ----------- | ----------- | ---------- |
| 101         | 2           | 1          |
| 102         | 2           | 1          |
| 103         | 3           | 1          |
| 104         | 3           | 0          |
| 105         | 0           | 1          |

---
**Query #6** <br>
What was the maximum number of pizzas delivered in a single order?
```sql
    WITH max_pizzas AS (
    SELECT order_id, 
      	   COUNT(pizza_id) as Total
    FROM new_customer_orders
    GROUP BY order_id
    ORDER BY order_id ASC
    )
    SELECT MAX(Total) as Max_Num_of_Pizzas
    FROM max_pizzas
    JOIN new_runner_orders ON 
    max_pizzas.order_id = new_runner_orders.order_id
    WHERE new_runner_orders.cancellation IS NULL;
```
| max_num_of_pizzas |
| ----------------- |
| 3                 |

---
**Query #7** <br>
For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
    SELECT customer_id,
    	   SUM(CASE WHEN exclusions IS NOT NULL 
               OR extras IS NOT NULL THEN 1 ELSE 0 END) AS HAS_CHANGES,
           SUM(CASE WHEN exclusions IS NULL
               AND extras IS NULL THEN 1 ELSE 0 END) AS NO_CHANGES
    FROM new_customer_orders 
    JOIN new_runner_orders ON 
    new_customer_orders.order_id = new_runner_orders.order_id
    WHERE cancellation IS NULL
    GROUP by customer_id;
```
| customer_id | has_changes | no_changes |
| ----------- | ----------- | ---------- |
| 101         | 0           | 2          |
| 102         | 0           | 3          |
| 103         | 3           | 0          |
| 104         | 2           | 1          |
| 105         | 1           | 0          |

---
**Query #8** <br>
How many pizzas were delivered that had both exclusions and extras?
```sql
    SELECT SUM(CASE WHEN exclusions IS NOT NULL 
               AND extras IS NOT NULL THEN 1 ELSE 0 END) as Total
    FROM new_customer_orders 
    JOIN new_runner_orders ON 
    new_customer_orders.order_id = new_runner_orders.order_id
    WHERE new_runner_orders.cancellation IS NULL;
```
| total |
| ----- |
| 1     |

---
**Query #9** <br>
What was the total volume of pizzas ordered for each hour of the day?
```sql
    SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day,
    	   COUNT(*) AS Total_Pizzas
    FROM new_customer_orders
    GROUP BY hour_of_day
    ORDER BY hour_of_day;
```
| hour_of_day | total_pizzas |
| ----------- | ------------ |
| 11          | 1            |
| 13          | 3            |
| 18          | 3            |
| 19          | 1            |
| 21          | 3            |
| 23          | 3            |

---
**Query #10** <br>
What was the volume of orders for each day of the week?
```sql
    SELECT to_char(order_time,'Day') as week_day,
    	   COUNT(*) as Total_Pizzas
    FROM new_customer_orders
    GROUP BY week_day;
```
| week_day  | total_pizzas |
| --------- | ------------ |
| Saturday  | 5            |
| Thursday  | 3            |
| Friday    | 1            |
| Wednesday | 5            |
</details>

---

<details>
<summary>
B. Runner and Customer Experience
</summary>

## B. Runner and Customer Experience 

**Query #1** <br>
How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
    With signups AS (
    SELECT runner_id,
    	     registration_date,
    	     registration_date - ((registration_date - '2021-01-01') % 7) AS starting_week
    FROM pizza_runner.runners
    )
    SELECT starting_week,
           COUNT(runner_id) AS Total_Runners
    FROM signups
    GROUP BY starting_week
    ORDER BY starting_week;
```
| starting_week            | total_runners |
| ------------------------ | ------------- |
| 2021-01-01T00:00:00.000Z | 2             |
| 2021-01-08T00:00:00.000Z | 1             |
| 2021-01-15T00:00:00.000Z | 1             |

---
**Query #2** <br>
What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
    With arrival AS (
    SELECT r.runner_id,
    	     r.pickup_time,
           (r.pickup_time - c.order_time) AS arrival_time
    FROM new_runner_orders AS r 
    JOIN new_customer_orders AS c ON
    r.order_id = c.order_id
    )
    SELECT runner_id,
    	     EXTRACT('minutes' FROM AVG(arrival_time)) AS avg_arrival_time
    FROM arrival
    GROUP BY runner_id
    ORDER BY runner_id;
```
| runner_id | avg_arrival_time |
| --------- | ---------------- |
| 1         | 15               |
| 2         | 23               |
| 3         | 10               |

---
**Query #3** <br>
Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
    With num_of_pizzas AS (
    SELECT order_id,
      	   order_time,
           COUNT(pizza_id) AS n_pizzas
    FROM new_customer_orders
    GROUP BY order_id, order_time
    ORDER BY order_id
    ),
    prep_time AS (
    SELECT r.runner_id,
    	     r.pickup_time,
           n.order_time,
    	     n.n_pizzas,
    	     (r.pickup_time - n.order_time) AS runner_arrival_time
    FROM new_runner_orders AS r 
    JOIN num_of_pizzas AS n ON
    r.order_id = n.order_id
    WHERE r.pickup_time IS NOT null
    )
    SELECT n_pizzas,
    	     AVG(runner_arrival_time) AS avg_order_time
    FROM prep_time
    GROUP BY n_pizzas
    ORDER BY n_pizzas ASC;
```
| n_pizzas | avg_order_time  |
| -------- | --------------- |
| 1        | 00:12:21.4 |
| 2        | 00:18:22.5 |
| 3        | 00:29:17   |

---
**Query #4** <br>
What was the average distance travelled for each customer?
```sql
    SELECT c.customer_id,
           ROUND(AVG(r.distance),2) AS avg_distance
    FROM new_customer_orders AS c
    JOIN new_runner_orders AS r ON
    c.order_id=r.order_id
    GROUP BY customer_id
    ORDER BY customer_id;
```
| customer_id | avg_distance |
| ----------- | ------------ |
| 101         | 20.00        |
| 102         | 16.73        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

---
**Query #5** <br>
What was the difference between the longest and shortest delivery times for all orders?
```sql
    SELECT MIN(duration) AS shortest_delivery,
    	     MAX(duration) AS longest_delivery,
           MAX(duration) - MIN(duration) AS difference
    FROM new_runner_orders;
```
| shortest_delivery | longest_delivery | difference |
| ----------------- | ---------------- | ---------- |
| 10                | 40               | 30         |

---
**Query #6** <br>
What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
    SELECT c.customer_id,
    	     r.order_id,
    	     r.runner_id,
           r.distance,
           r.duration,
    	     ROUND((60 * r.distance / r.duration), 2) AS runner_speed_km_per_hr
    FROM new_customer_orders as c
    JOIN new_runner_orders AS r ON
    c.order_id = r.order_id
    WHERE r.pickup_time IS NOT NULL
    GROUP BY c.customer_id, r.order_id, r.runner_id, r.distance, r.duration, runner_speed_km_per_hr
    ORDER BY runner_speed_km_per_hr DESC;
```
| customer_id | order_id | runner_id | distance | duration | runner_speed_km_per_hr |
| ----------- | -------- | --------- | -------- | -------- | ---------------------- |
| 102         | 8        | 2         | 23.4     | 15       | 93.60                  |
| 104         | 10       | 1         | 10       | 10       | 60.00                  |
| 105         | 7        | 2         | 25       | 25       | 60.00                  |
| 101         | 2        | 1         | 20       | 27       | 44.44                  |
| 102         | 3        | 1         | 13.4     | 20       | 40.20                  |
| 104         | 5        | 3         | 10       | 15       | 40.00                  |
| 101         | 1        | 1         | 20       | 32       | 37.50                  |
| 103         | 4        | 2         | 23.4     | 40       | 35.10                  |

---
**Query #7** <br>
What is the successful delivery percentage for each runner?
```sql
    SELECT runner_id,
    	   COUNT(order_id) AS total_orders,
    	   COUNT(pickup_time) AS completed_delivery,
    	   CONCAT(ROUND(100 * COUNT(pickup_time) / COUNT(order_id)),'%') AS success_rate
    FROM new_runner_orders
    GROUP BY runner_id
    ORDER BY runner_id;
```
| runner_id | total_orders | completed_delivery | success_rate |
| --------- | ------------ | ------------------ | ------------ |
| 1         | 4            | 4                  | 100%         |
| 2         | 4            | 3                  | 75%          |
| 3         | 2            | 1                  | 50%          |

</details>

---

<details>
<summary>
C. Ingredient Optimisation
</summary>
</details>

--- 

<details>
<summary>
D. Pricing and Ratings
</summary>
</details>

---

<details>
<summary>
E. Bonus Questions
</summary>
</details>
