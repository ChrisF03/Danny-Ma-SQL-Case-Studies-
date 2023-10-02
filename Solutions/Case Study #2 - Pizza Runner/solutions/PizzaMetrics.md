## A. Pizza Metrics 🍕

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
