## D. Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

## Solution:

For this challenge, I propose that Danny could use any of the completed queries to find any information he needs for his desired month by just appending the following line of sql code:

```sql
WHERE EXTRACT(MONTH FROM start_txn_time) = -- (insert desired month number here)
```
### Example: 
**Query #1** <br>
In this query we retrieved the highest grossing products overall for the entire timeperiod covered by the datasets
```sql
    SELECT t1.product_name,
           TO_CHAR(SUM(t2.price * t2.qty), '9,999,999') as gross_revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2 ON
    t1.product_id = t2.prod_id
    GROUP BY t1.product_name
    ORDER BY gross_revenue DESC 
    LIMIT 3;
```
| product_name                 | gross_revenue |
| ---------------------------- | ------------- |
| Blue Polo Shirt - Mens       |    217,683    |
| Grey Fashion Jacket - Womens |    209,304    |
| White Tee Shirt - Mens       |    152,000    |

**Query #2** <br>
Now we add the line of code suggested at the top after the JOIN and now we can filter the output by month. Here we are looking at the revenues for the month of January ONLY.
```sql
    SELECT t1.product_name,
           TO_CHAR(SUM(t2.price * t2.qty), '9,999,999') as gross_revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2 ON
    t1.product_id = t2.prod_id
    WHERE EXTRACT(MONTH FROM t2.start_txn_time) = 1
    GROUP BY t1.product_name
    ORDER BY gross_revenue DESC 
    LIMIT 3;
```
| product_name                 | gross_revenue |
| ---------------------------- | ------------- |
| Grey Fashion Jacket - Womens |     70,200    |
| Blue Polo Shirt - Mens       |     69,198    |
| White Tee Shirt - Mens       |     50,240    |
