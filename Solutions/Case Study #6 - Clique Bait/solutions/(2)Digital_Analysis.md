## 2. Digital Analysis

**Query #1** <br>
How many users are there?
```sql
    SELECT COUNT(DISTINCT user_id) as users
    FROM clique_bait.users;
```
| users |
| ----- |
| 500   |

---
**Query #2** <br>
How many cookies does each user have on average?
```sql
    WITH all_cookies AS (
    SELECT user_id,
      	   COUNT(cookie_id) as cookies
    FROM clique_bait.users
    GROUP BY user_id
    )
    SELECT ROUND(AVG(cookies)) as avg_cookies
    FROM all_cookies;
```
| avg_cookies |
| ----------- |
| 4           |

---
**Query #3** <br>
What is the unique number of visits by all users per month?
```sql
    SELECT EXTRACT(MONTH FROM event_time) AS month, 
      	   COUNT(DISTINCT visit_id) AS total_visits
    FROM clique_bait.events
    GROUP BY EXTRACT(MONTH FROM event_time);
```
| month | total_visits |
| ----- | ------------ |
| 1     | 876          |
| 2     | 1488         |
| 3     | 916          |
| 4     | 248          |
| 5     | 36           |

---
**Query #4** <br>
What is the number of events for each event type?
```sql
    SELECT t1.event_type,
    	   t1.event_name,
    	   COUNT(t2.visit_id) as total
    FROM clique_bait.event_identifier as t1
    JOIN clique_bait.events as t2 ON
    t1.event_type = t2.event_type
    GROUP BY t1.event_type, t1.event_name
    ORDER BY total DESC;
```
| event_type | event_name    | total |
| ---------- | ------------- | ----- |
| 1          | Page View     | 20928 |
| 2          | Add to Cart   | 8451  |
| 3          | Purchase      | 1777  |
| 4          | Ad Impression | 876   |
| 5          | Ad Click      | 702   |

---
**Query #5** <br>
What is the percentage of visits which have a purchase event?
```sql
    SELECT ROUND(100 * SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END)
                 / COUNT(DISTINCT visit_id), 2) as purchase_percentage
    FROM clique_bait.events;
```
| purchase_percentage |
| ------------------- |
| 49.00               |

---
**Query #6** <br>
What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
    WITH counts AS (
    SELECT visit_id,
    	   SUM(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) as viewed_checkout,
           SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) as purchased
    FROM clique_bait.events
    GROUP BY visit_id
    )
    SELECT SUM(viewed_checkout) as viewed_checkout,
    	   SUM(purchased) as purchased,
           ROUND(100 * (1 - SUM(purchased) / SUM(viewed_checkout)), 2) AS visit_percentage
    FROM counts;
```
| viewed_checkout | purchased | visit_percentage |
| --------------- | --------- | ---------------- |
| 2103            | 1777      | 15.50            |

---
**Query #7** <br>
What are the top 3 pages by number of views?
```sql
    SELECT t1.page_name,
    	   COUNT(t2.visit_id) as total_views
    FROM clique_bait.page_hierarchy as t1
    JOIN clique_bait.events as t2 ON
    t1.page_id = t2.page_id
    WHERE t2.event_type = 1
    GROUP BY t1.page_name
    ORDER BY total_views DESC
    LIMIT 3;
```
| page_name    | total_views |
| ------------ | ----------- |
| All Products | 3174        |
| Checkout     | 2103        |
| Home Page    | 1782        |

---
**Query #8** <br>
What is the number of views and cart adds for each product category?
```sql
    SELECT t1.product_category,
    	   SUM(CASE WHEN t2.event_type = 1 THEN 1 ELSE 0 END) as viewed,
           SUM(CASE WHEN t2.event_type = 2 THEN 1 ELSE 0 END) as carted
    FROM clique_bait.page_hierarchy as t1
    JOIN clique_bait.events as t2 ON
    t1.page_id = t2.page_id
    WHERE t1.product_category IS NOT NULL
    GROUP BY t1.product_category
    ORDER BY carted DESC;
```
| product_category | viewed | carted |
| ---------------- | ------ | ------ |
| Shellfish        | 6204   | 3792   |
| Fish             | 4633   | 2789   |
| Luxury           | 3032   | 1870   |

---
**Query #9** <br>
What are the top 3 products by purchases?
```sql
    WITH purchases AS (
    SELECT visit_id
    FROM clique_bait.events
    WHERE event_type = 3
    GROUP BY visit_id
    )
    SELECT t1.page_name,
    	   COUNT(t2.*) as purchases
    FROM clique_bait.page_hierarchy as t1
    JOIN clique_bait.events as t2 ON
    t1.page_id = t2.page_id
    WHERE t1.product_category IS NOT NULL 
    AND t2.event_type = (SELECT event_type 
                         FROM clique_bait.event_identifier
                         WHERE event_name LIKE 'Add%')
    AND t2.visit_id IN (SELECT visit_id 
                        FROM purchases)
    GROUP BY t1.page_name
    ORDER BY purchases DESC
    LIMIT 3;
```
| page_name | purchases |
| --------- | --------- |
| Lobster   | 754       |
| Oyster    | 726       |
| Crab      | 719       |
