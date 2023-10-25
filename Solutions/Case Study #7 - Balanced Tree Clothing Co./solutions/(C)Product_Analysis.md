## C. Product Analysis

**Query #1** <br>
What are the top 3 products by total revenue before discount?
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

---
**Query #2** <br>
What is the total quantity, revenue and discount for each segment?
```sql
    SELECT t1.segment_id,
    	   t1.segment_name,
           TO_CHAR(SUM(t2.qty), '9,999,999') as quantity,
           TO_CHAR(SUM(t2.price * t2.qty), '9,999,999') as revenue,
           TO_CHAR(ROUND(SUM((t2.price * t2.qty) * (t2.discount::NUMERIC / 100)), 2),'9,999,999') AS total_discounts,
           TO_CHAR(ROUND(SUM((t2.price * t2.qty) * (1 - discount::NUMERIC / 100)), 2), '9,999,999') AS total_revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2
    ON t1.product_id = t2.prod_id
    GROUP BY t1.segment_id, t1.segment_name
    ORDER BY t1.segment_id;
```
| segment_id | segment_name | quantity   | revenue    | total_discounts | total_revenue |
| ---------- | ------------ | ---------- | ---------- | --------------- | ------------- |
| 3          | Jeans        |     11,349 |    208,350 |     25,344      |    183,006    |
| 4          | Jacket       |     11,385 |    366,983 |     44,277      |    322,706    |
| 5          | Shirt        |     11,265 |    406,143 |     49,594      |    356,549    |
| 6          | Socks        |     11,217 |    307,977 |     37,013      |    270,964    |

---
**Query #3** <br>
What is the top selling product for each segment?
```sql
    WITH prod_qty AS (
    SELECT t1.prod_id,
    	   t2.product_name,
           t2.segment_id,
           t2.segment_name,
           SUM(t1.qty) as qty,
           ROW_NUMBER() OVER (PARTITION BY t2.segment_id
                              ORDER BY SUM(t1.qty) DESC) as rank
    FROM balanced_tree.sales as t1 
    JOIN balanced_tree.product_details as t2
    ON t1.prod_id = t2.product_id
    GROUP BY t1.prod_id, t2.product_name, t2.segment_id, t2.segment_name
    )
    SELECT segment_id,
    	   segment_name,
           product_name as best_seller,
           qty
    FROM prod_qty 
    WHERE rank = 1 
    GROUP BY segment_id, segment_name, product_name, qty;
```
| segment_id | segment_name | best_seller                   | qty  |
| ---------- | ------------ | ----------------------------- | ---- |
| 3          | Jeans        | Navy Oversized Jeans - Womens | 3856 |
| 4          | Jacket       | Grey Fashion Jacket - Womens  | 3876 |
| 5          | Shirt        | Blue Polo Shirt - Mens        | 3819 |
| 6          | Socks        | Navy Solid Socks - Mens       | 3792 |

---
**Query #4** <br>
What is the total quantity, revenue and discount for each category?
```sql
    SELECT t1.category_id,
    	   t1.category_name,
           TO_CHAR(SUM(t2.qty), '9,999,999') as quantity,
           TO_CHAR(SUM(t2.price * t2.qty), '9,999,999') as revenue,
           TO_CHAR(ROUND(SUM((t2.price * t2.qty) * (t2.discount::NUMERIC / 100)), 2),'9,999,999') AS total_discounts,
           TO_CHAR(ROUND(SUM((t2.price * t2.qty) * (1 - discount::NUMERIC / 100)), 2), '9,999,999') AS total_revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2
    ON t1.product_id = t2.prod_id
    GROUP BY t1.category_id, t1.category_name
    ORDER BY t1.category_id;
```
| category_id | category_name | quantity   | revenue    | total_discounts | total_revenue |
| ----------- | ------------- | ---------- | ---------- | --------------- | ------------- |
| 1           | Womens        |     22,734 |    575,333 |     69,621      |    505,712    |
| 2           | Mens          |     22,482 |    714,120 |     86,608      |    627,512    |

---
**Query #5** <br>
What is the top selling product for each category?
```sql
    WITH cat_qty AS (
    SELECT t1.prod_id,
    	   t2.product_name,
           t2.category_id,
           t2.category_name,
           SUM(t1.qty) as qty,
           ROW_NUMBER() OVER (PARTITION BY t2.category_id
                              ORDER BY SUM(t1.qty) DESC) as rank
    FROM balanced_tree.sales as t1 
    JOIN balanced_tree.product_details as t2
    ON t1.prod_id = t2.product_id
    GROUP BY t1.prod_id, t2.product_name, t2.category_id, t2.category_name
    )
    SELECT category_id,
    	   category_name,
           product_name as best_seller,
           qty
    FROM cat_qty 
    WHERE rank = 1 
    GROUP BY category_id, category_name, product_name, qty;
```
| category_id | category_name | best_seller                  | qty  |
| ----------- | ------------- | ---------------------------- | ---- |
| 1           | Womens        | Grey Fashion Jacket - Womens | 3876 |
| 2           | Mens          | Blue Polo Shirt - Mens       | 3819 |

---
**Query #6** <br>
What is the percentage split of revenue by product for each segment?
```sql
    WITH prod_rev AS (
    SELECT t1.prod_id,
    	   t2.product_name,
           t2.segment_id,
           t2.segment_name,
           SUM(t1.qty * t1.price) as revenue
    FROM balanced_tree.sales as t1 
    JOIN balanced_tree.product_details as t2
    ON t1.prod_id = t2.product_id
    GROUP BY t1.prod_id, t2.product_name, t2.segment_id, t2.segment_name
    )
    SELECT segment_id,
    	   segment_name,
           product_name,
           TO_CHAR(revenue, '9,999,999') as revenue,
           ROUND(100 * (revenue / SUM(revenue) OVER (
      	PARTITION BY segment_id)), 2) AS revenue_percentage
    FROM prod_rev
    GROUP BY segment_id, segment_name, product_name, revenue
    ORDER BY segment_id, revenue_percentage DESC;
```
| segment_id | segment_name | product_name                     | revenue    | revenue_percentage |
| ---------- | ------------ | -------------------------------- | ---------- | ------------------ |
| 3          | Jeans        | Black Straight Jeans - Womens    |    121,152 | 58.15              |
| 3          | Jeans        | Navy Oversized Jeans - Womens    |     50,128 | 24.06              |
| 3          | Jeans        | Cream Relaxed Jeans - Womens     |     37,070 | 17.79              |
| 4          | Jacket       | Grey Fashion Jacket - Womens     |    209,304 | 57.03              |
| 4          | Jacket       | Khaki Suit Jacket - Womens       |     86,296 | 23.51              |
| 4          | Jacket       | Indigo Rain Jacket - Womens      |     71,383 | 19.45              |
| 5          | Shirt        | Blue Polo Shirt - Mens           |    217,683 | 53.60              |
| 5          | Shirt        | White Tee Shirt - Mens           |    152,000 | 37.43              |
| 5          | Shirt        | Teal Button Up Shirt - Mens      |     36,460 | 8.98               |
| 6          | Socks        | Navy Solid Socks - Mens          |    136,512 | 44.33              |
| 6          | Socks        | Pink Fluro Polkadot Socks - Mens |    109,330 | 35.50              |
| 6          | Socks        | White Striped Socks - Mens       |     62,135 | 20.18              |

---
**Query #7** <br>
What is the percentage split of revenue by segment for each category?
```sql
    WITH cat_rev AS (
    SELECT t1.segment_id,
    	   t1.segment_name,
           t1.category_id,
           t1.category_name,
           SUM(t2.qty * t2.price) as revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2
    ON t1.product_id = t2.prod_id
    GROUP BY t1.segment_id, t1.segment_name, t1.category_id, t1.category_name
    )
    SELECT category_id,
    	   category_name,
           segment_name,
           TO_CHAR(revenue, '9,999,999') as revenue,
           ROUND(100 * (revenue / SUM(revenue) OVER (
      	PARTITION BY category_id)), 2) AS revenue_percentage
    FROM cat_rev
    GROUP BY category_id, category_name, segment_name, revenue;
```
| category_id | category_name | segment_name | revenue    | revenue_percentage |
| ----------- | ------------- | ------------ | ---------- | ------------------ |
| 1           | Womens        | Jacket       |    366,983 | 63.79              |
| 1           | Womens        | Jeans        |    208,350 | 36.21              |
| 2           | Mens          | Shirt        |    406,143 | 56.87              |
| 2           | Mens          | Socks        |    307,977 | 43.13              |

---
**Query #8** <br>
What is the percentage split of total revenue by category?
```sql
    WITH total_cat_rev AS (
    SELECT t1.category_id,
    	   t1.category_name,
           SUM(t2.qty * t2.price) as revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2
    ON t1.product_id = t2.prod_id
    GROUP BY t1.category_id, t1.category_name
    )
    SELECT category_name,
    	   TO_CHAR(revenue, '9,999,999') as revenue,
           ROUND(100 * (revenue / SUM(revenue) OVER()), 2) as percent_of_total_revenue
    FROM total_cat_rev
    GROUP BY category_name, revenue;
```
| category_name | revenue    | percent_of_total_revenue |
| ------------- | ---------- | ------------------------ |
| Mens          |    714,120 | 55.38                    |
| Womens        |    575,333 | 44.62                    |

---
**Query #9** <br>
What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql
    WITH penetration AS (
    SELECT t1.product_id,
      	   t1.product_name,
      	   COUNT(DISTINCT t2.txn_id) AS n_sold,
      	   (SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales) AS total_transactions
    FROM balanced_tree.product_details AS t1
    JOIN balanced_tree.sales AS t2 
    ON t1.product_id = t2.prod_id
    GROUP BY t1.product_id, t1.product_name
    )
    SELECT product_name, 
           n_sold AS number_of_items_sold,
      	   ROUND(100 * (n_sold::NUMERIC / total_transactions), 2) AS product_penetration
    FROM penetration
    GROUP BY product_name, number_of_items_sold, product_penetration
    ORDER BY product_penetration DESC;
```
| product_name                     | number_of_items_sold | product_penetration |
| -------------------------------- | -------------------- | ------------------- |
| Navy Solid Socks - Mens          | 1281                 | 51.24               |
| Grey Fashion Jacket - Womens     | 1275                 | 51.00               |
| Navy Oversized Jeans - Womens    | 1274                 | 50.96               |
| Blue Polo Shirt - Mens           | 1268                 | 50.72               |
| White Tee Shirt - Mens           | 1268                 | 50.72               |
| Pink Fluro Polkadot Socks - Mens | 1258                 | 50.32               |
| Indigo Rain Jacket - Womens      | 1250                 | 50.00               |
| Khaki Suit Jacket - Womens       | 1247                 | 49.88               |
| Black Straight Jeans - Womens    | 1246                 | 49.84               |
| Cream Relaxed Jeans - Womens     | 1243                 | 49.72               |
| White Striped Socks - Mens       | 1243                 | 49.72               |
| Teal Button Up Shirt - Mens      | 1242                 | 49.68               |

---
**Query #10** <br>
What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```diff
- INCOMPLETE
```
