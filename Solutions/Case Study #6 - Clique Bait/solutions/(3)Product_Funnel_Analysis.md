## 3. Product Funnel Analysis

Using a single SQL query - create a new output table which has the following details:

1) How many times was each product viewed?
2) How many times was each product added to cart?
3) How many times was each product added to a cart but not purchased (abandoned)?
4) How many times was each product purchased?

```sql
    CREATE TEMP TABLE product_info AS (
    WITH viewed_carted AS (
    SELECT t2.visit_id,
    	   t2.page_id,
    	   t1.page_name as product,
           SUM(CASE WHEN t2.event_type = 1 THEN 1 ELSE 0 END) as viewed,
           SUM(CASE WHEN t2.event_type = 2 THEN 1 ELSE 0 END) as carted
    FROM clique_bait.page_hierarchy as t1
    JOIN clique_bait.events as t2 ON
    t1.page_id = t2.page_id
    WHERE t1.product_category IS NOT NULL
    GROUP BY t2.visit_id, t2.page_id, t1.page_name
    ),
    purchased AS (
    SELECT visit_id
    FROM clique_bait.events AS e1
    JOIN clique_bait.event_identifier AS e2 ON 
    e2.event_type = e1.event_type
    WHERE e2.event_name = 'Purchase'
    ),
    combined AS (
    SELECT t1.*, COUNT(t2.*) as purchased
    FROM viewed_carted as t1
    LEFT JOIN purchased as t2 ON
    t1.visit_id = t2.visit_id
    GROUP BY t1.visit_id, t1.page_id, t1.product, viewed, carted
    )
    SELECT product,
    	   SUM(viewed) as viewed,
           SUM(carted) as carted,
           SUM(CASE WHEN carted = 1 AND purchased = 1 THEN 1 ELSE 0 END) AS purchased,
           SUM(CASE WHEN carted = 1 AND purchased = 0 THEN 1 ELSE 0 END) AS abandoned
    FROM combined
    GROUP BY product
    );
```
```sql
    SELECT * FROM product_info;
```
| product        | viewed | carted | purchased | abandoned |
| -------------- | ------ | ------ | --------- | --------- |
| Abalone        | 1525   | 932    | 699       | 233       |
| Oyster         | 1568   | 943    | 726       | 217       |
| Salmon         | 1559   | 938    | 711       | 227       |
| Crab           | 1564   | 949    | 719       | 230       |
| Tuna           | 1515   | 931    | 697       | 234       |
| Lobster        | 1547   | 968    | 754       | 214       |
| Kingfish       | 1559   | 920    | 707       | 213       |
| Russian Caviar | 1563   | 946    | 697       | 249       |
| Black Truffle  | 1469   | 924    | 707       | 217       |

---
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
```sql
    CREATE TEMP TABLE category_info AS (
    WITH viewed_carted AS (
    SELECT t2.visit_id,
    	   t2.page_id,
    	   t1.product_category as category,
           SUM(CASE WHEN t2.event_type = 1 THEN 1 ELSE 0 END) as viewed,
           SUM(CASE WHEN t2.event_type = 2 THEN 1 ELSE 0 END) as carted
    FROM clique_bait.page_hierarchy as t1
    JOIN clique_bait.events as t2 ON
    t1.page_id = t2.page_id
    WHERE t1.product_category IS NOT NULL
    GROUP BY t2.visit_id, t2.page_id, t1.product_category
    ),
    purchased AS (
    SELECT visit_id
    FROM clique_bait.events AS e1
    JOIN clique_bait.event_identifier AS e2 ON 
    e2.event_type = e1.event_type
    WHERE e2.event_name = 'Purchase'
    ),
    combined AS (
    SELECT t1.*, COUNT(t2.*) as purchased
    FROM viewed_carted as t1
    LEFT JOIN purchased as t2 ON
    t1.visit_id = t2.visit_id
    GROUP BY t1.visit_id, t1.page_id, category, viewed, carted
    )
    SELECT category,
    	   SUM(viewed) as viewed,
           SUM(carted) as carted,
           SUM(CASE WHEN carted = 1 AND purchased = 1 THEN 1 ELSE 0 END) AS purchased,
           SUM(CASE WHEN carted = 1 AND purchased = 0 THEN 1 ELSE 0 END) AS abandoned
    FROM combined
    GROUP BY category
    );
```
```sql
    SELECT * FROM category_info;
```
| category  | viewed | carted | purchased | abandoned |
| --------- | ------ | ------ | --------- | --------- |
| Luxury    | 3032   | 1870   | 1404      | 466       |
| Shellfish | 6204   | 3792   | 2898      | 894       |
| Fish      | 4633   | 2789   | 2115      | 674       |

---
Using your 2 new output tables - answer the following questions:

Which product had the most views, cart adds and purchases?

* Oyster had the most views with 1,568.
* Lobster had the most cart adds with 968.
* Lobster also had the most purchases with 754.
---
Which product was most likely to be abandoned?

* Russian Caviar was abandoned the most with a total of 249.
---
Which product had the highest view to purchase percentage?
```sql
    SELECT product,
    	   ROUND(100 * (purchased/viewed), 2) AS view_to_purchase_ratio
    FROM product_info
    GROUP BY product, viewed, purchased
    ORDER BY view_to_purchase_ratio DESC
    LIMIT 1;
```
| product | view_to_purchase_ratio |
| ------- | ---------------------- |
| Lobster | 48.74                  |

---
What is the average conversion rate from view to cart add?
```sql
    SELECT ROUND(100 * AVG(carted/viewed), 2) as view_to_cart_ratio
    FROM product_info;
```
| view_to_cart_ratio |
| ------------------ |
| 60.95              |

---
`What is the average conversion rate from cart add to purchase?`
```sql
    SELECT ROUND(100 * AVG(purchased/carted), 2) as cart_to_purchase_ratio
    FROM product_info;
```
| cart_to_purchase_ratio |
| ---------------------- |
| 75.93                  |
