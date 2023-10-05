## D. Pricing and Ratings 💲

**Query #1** <br>
If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
    SELECT SUM(CASE
                   WHEN pizza_id = 1 THEN 12 
                   WHEN pizza_id = 2 THEN 10
               	   END) as income
    FROM new_customer_orders
    JOIN new_runner_orders ON
    new_customer_orders.order_id = new_runner_orders.order_id
    WHERE new_runner_orders.cancellation IS NULL;
```
| income |
| ------ |
| 138    |

---
**Query #2** <br>
What if there was an additional $1 charge for any pizza extras?
```sql
    With each_extra AS (
    SELECT
      new_customer_orders.order_id,
      UNNEST(STRING_TO_ARRAY(extras, ','))::NUMERIC AS extras
    FROM new_customer_orders
    ),

    extra_count AS (
    SELECT order_id,
      	   COUNT(extras) as total
    FROM each_extra
    GROUP BY order_id
    ),

    totals AS (
    SELECT t1.order_id,
      	   t1.pizza_id,
      	   SUM(CASE
                   WHEN pizza_id = 1 THEN 12
                   WHEN pizza_id = 2 THEN 10
                   END) AS total_price,
      		t3.total
    FROM new_customer_orders AS t1
    JOIN new_runner_orders AS t2 
    ON t2.order_id = t1.order_id
    LEFT JOIN extra_count AS t3
    ON t3.order_id = t1.order_id
    WHERE t2.cancellation IS NULL
    GROUP BY t1.order_id, t1.pizza_id, t3.total
    )

    SELECT SUM(total_price) + SUM(total) AS total_income
    FROM  totals;
```
| total_income |
| ------------ |
| 142          |

---
**Query #3** <br>
The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
    DROP TABLE IF EXISTS rating;
    CREATE TABLE ratings (
      "order_id" INTEGER,
      "customer_id" INTEGER,
      "order_time" TIMESTAMP,
      "runner_id" INTEGER,
      "rating" INTEGER);
```
```sql
    INSERT INTO ratings
    SELECT t1.order_id,
    	   t1.customer_id,
           t1.order_time,
           t2.runner_id,
           FLOOR(1 + 5 * random()) AS rating
    FROM new_customer_orders as t1
    JOIN new_runner_orders as t2 ON
    t1.order_id = t2.order_id
    WHERE t2.pickup_time IS NOT NULL;

    SELECT * FROM ratings;
```
| order_id | customer_id | order_time               | runner_id | rating |
| -------- | ----------- | ------------------------ | --------- | ------ |
| 1        | 101         | 2020-01-01T18:05:02.000Z | 1         | 5      |
| 2        | 101         | 2020-01-01T19:00:52.000Z | 1         | 5      |
| 3        | 102         | 2020-01-02T23:51:23.000Z | 1         | 3      |
| 3        | 102         | 2020-01-02T23:51:23.000Z | 1         | 3      |
| 4        | 103         | 2020-01-04T13:23:46.000Z | 2         | 1      |
| 4        | 103         | 2020-01-04T13:23:46.000Z | 2         | 1      |
| 4        | 103         | 2020-01-04T13:23:46.000Z | 2         | 3      |
| 5        | 104         | 2020-01-08T21:00:29.000Z | 3         | 5      |
| 7        | 105         | 2020-01-08T21:20:29.000Z | 2         | 3      |
| 8        | 102         | 2020-01-09T23:54:33.000Z | 2         | 3      |
| 10       | 104         | 2020-01-11T18:34:49.000Z | 1         | 1      |
| 10       | 104         | 2020-01-11T18:34:49.000Z | 1         | 1      |

---
**Query #4** <br>
Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas
```sql
    SELECT t1.customer_id,
    	   t1.order_id,
           t1.runner_id,
           t1.rating,
           t1.order_time,
           t2.pickup_time,
           (t2.pickup_time::TIMESTAMP - t1.order_time::TIMESTAMP) AS time_diff,
           t2.duration,
           ROUND(60 * t2.distance / t2.duration, 2) AS avg_speed_kph,
           COUNT(t2.pickup_time) AS total_delivered
    FROM ratings AS t1
    JOIN new_runner_orders AS t2 ON 
    t1.order_id = t2.order_id
    GROUP BY t1.customer_id, t1.order_id, t1.runner_id, t1.rating, 					t1.order_time, t2.pickup_time, t2.duration, t2.distance
    ORDER BY t1.order_id;
```
| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | time_diff       | duration | avg_speed_kph | total_delivered |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | --------------- | -------- | ------------- | --------------- |
| 101         | 1        | 1         | 5      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | [object Object] | 32       | 37.50         | 1               |
| 101         | 2        | 1         | 5      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | [object Object] | 27       | 44.44         | 1               |
| 102         | 3        | 1         | 3      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | [object Object] | 20       | 40.20         | 2               |
| 103         | 4        | 2         | 1      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | [object Object] | 40       | 35.10         | 2               |
| 103         | 4        | 2         | 3      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | [object Object] | 40       | 35.10         | 1               |
| 104         | 5        | 3         | 5      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | [object Object] | 15       | 40.00         | 1               |
| 105         | 7        | 2         | 3      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | [object Object] | 25       | 60.00         | 1               |
| 102         | 8        | 2         | 3      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | [object Object] | 15       | 93.60         | 1               |
| 104         | 10       | 1         | 1      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | [object Object] | 10       | 60.00         | 2               |

---
**Query #5** <br>
If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
    With revenue AS(
    SELECT SUM(CASE
                   WHEN pizza_id = 1 THEN 12 
                   WHEN pizza_id = 2 THEN 10
               	   END) as income
    FROM new_customer_orders
    JOIN new_runner_orders ON
    new_customer_orders.order_id = new_runner_orders.order_id
    WHERE new_runner_orders.cancellation IS NULL
    ),
    payouts AS (
    SELECT (SUM(distance) * .30) AS payout
    FROM new_runner_orders
    WHERE pickup_time IS NOT NULL
    )
    SELECT ROUND((t1.income - t2.payout), 2) as profit
    FROM revenue as t1, payouts as t2;
```
| profit |
| ------ |
| 94.44  |
