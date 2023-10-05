## C. Ingredient Optimization :cheese:

**Query #1** <br>
What are the standard ingredients for each pizza?
```sql
    WITH toppings AS (
    SELECT pizza_names.pizza_id,
      	   pizza_names.pizza_name,
           UNNEST(STRING_TO_ARRAY(pizza_recipes.toppings, ','))::NUMERIC AS topping
    FROM pizza_runner.pizza_names
    JOIN pizza_runner.pizza_recipes ON
    pizza_names.pizza_id = pizza_recipes.pizza_id
    GROUP by pizza_names.pizza_id, pizza_names.pizza_name, pizza_recipes.toppings
    ORDER BY pizza_names.pizza_id
    )
    SELECT pizza_id,
    	   pizza_name,
           STRING_AGG(pizza_toppings.topping_name, ',') AS toppings
    FROM toppings
    JOIN pizza_runner.pizza_toppings ON
    toppings.topping = pizza_toppings.topping_id
    GROUP BY pizza_id, pizza_name
    ORDER BY pizza_id;
```
| pizza_id | pizza_name | toppings                                                       |
| -------- | ---------- | -------------------------------------------------------------- |
| 1        | Meatlovers | BBQ Sauce,Pepperoni,Cheese,Salami,Chicken,Bacon,Mushrooms,Beef |
| 2        | Vegetarian | Tomato Sauce,Cheese,Mushrooms,Onions,Peppers,Tomatoes          |

---
**Query #2** <br>
What was the most commonly added extra?
```sql
    WITH extras_ordered AS (
    SELECT UNNEST(STRING_TO_ARRAY(new_customer_orders.extras, ','))::NUMERIC AS topping_id
    FROM new_customer_orders
    )
    SELECT pizza_toppings.topping_name AS most_common_topping, 
    	   COUNT(extras_ordered.topping_id) as Total
    FROM pizza_runner.pizza_toppings
    JOIN extras_ordered ON
    pizza_toppings.topping_id = extras_ordered.topping_id
    GROUP BY pizza_toppings.topping_name
    ORDER BY Total DESC
    LIMIT 1;
```
| most_common_topping | total |
| ------------------- | ----- |
| Bacon               | 4     |

---
**Query #3** <br>
What was the most common exclusion?
```sql
    WITH exclusions AS (
    SELECT UNNEST(STRING_TO_ARRAY(new_customer_orders.exclusions, ','))::NUMERIC AS topping_id
    FROM new_customer_orders
    )
    SELECT pizza_toppings.topping_name AS most_excluded_topping, 
    	   COUNT(exclusions.topping_id) as Total
    FROM pizza_runner.pizza_toppings
    JOIN exclusions ON
    pizza_toppings.topping_id = exclusions.topping_id
    GROUP BY pizza_toppings.topping_name
    ORDER BY Total DESC
    LIMIT 1;
```
| most_excluded_topping | total |
| --------------------- | ----- |
| Cheese                | 4     |

---
**Query #4**  <br> 
```diff 
-INCOMPLETE
```
Generate an order item for each record in the customers_orders table in the format of one of the following: <br>
	- Meat Lovers <br>
	- Meat Lovers - Exclude Beef <br>
	- Meat Lovers - Extra Bacon <br>
	- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```sql
    SELECT t1.order_id,
    	   t1.customer_id,
           t1.pizza_id,
           t1.order_time,
           CASE WHEN t1.exclusions IS NOT NULL AND t1.extras IS NULL
                  THEN CONCAT(t2.pizza_name, ' - ', 'Exclude: ', t1.exclusions)
                WHEN t1.exclusions IS NULL AND t1.extras IS NOT NULL
                  THEN CONCAT(t2.pizza_name, ' - ', 'Extra: ', t1.extras)
                WHEN t1.exclusions IS NOT NULL AND t1.extras IS NOT NULL
                  THEN CONCAT(t2.pizza_name, ' - ', 'Exclude: ', t1.exclusions, ' - ', 'Extra: ', t1.extras)
      	       ELSE t2.pizza_name
      	       END AS order_item
    FROM new_customer_orders AS t1
    JOIN pizza_runner.pizza_names AS t2 ON
    t1.pizza_id = t2.pizza_id
    ORDER BY t1.order_id ASC;
```
| order_id | customer_id | pizza_id | order_time               | order_item                               |
| -------- | ----------- | -------- | ------------------------ | ---------------------------------------- |
| 1        | 101         | 1        | 2020-01-01T18:05:02.000Z | Meatlovers                               |
| 2        | 101         | 1        | 2020-01-01T19:00:52.000Z | Meatlovers                               |
| 3        | 102         | 2        | 2020-01-02T23:51:23.000Z | Vegetarian                               |
| 3        | 102         | 1        | 2020-01-02T23:51:23.000Z | Meatlovers                               |
| 4        | 103         | 1        | 2020-01-04T13:23:46.000Z | Meatlovers - Exclude: 4                  |
| 4        | 103         | 1        | 2020-01-04T13:23:46.000Z | Meatlovers - Exclude: 4                  |
| 4        | 103         | 2        | 2020-01-04T13:23:46.000Z | Vegetarian - Exclude: 4                  |
| 5        | 104         | 1        | 2020-01-08T21:00:29.000Z | Meatlovers - Extra: 1                    |
| 6        | 101         | 2        | 2020-01-08T21:03:13.000Z | Vegetarian                               |
| 7        | 105         | 2        | 2020-01-08T21:20:29.000Z | Vegetarian - Extra: 1                    |
| 8        | 102         | 1        | 2020-01-09T23:54:33.000Z | Meatlovers                               |
| 9        | 103         | 1        | 2020-01-10T11:22:59.000Z | Meatlovers - Exclude: 4 - Extra: 1, 5    |
| 10       | 104         | 1        | 2020-01-11T18:34:49.000Z | Meatlovers                               |
| 10       | 104         | 1        | 2020-01-11T18:34:49.000Z | Meatlovers - Exclude: 2, 6 - Extra: 1, 4 |

---
**Query #5**  <br>
Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
	--For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```diff 
-INCOMPLETE
```
---
**Query #6**  <br>
What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```diff 
-INCOMPLETE
```
