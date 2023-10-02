## B. Runner and Customer Experience 🛵

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

- runner #2 is very fast. Maybe a little TOO fast ...
- customer 102 appears to be a big tipper.
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
