## B. Transaction Analysis

**Query #1** <br>
How many unique transactions were there?
```sql
    SELECT COUNT(DISTINCT txn_id)
    FROM balanced_tree.sales;
```
| count |
| ----- |
| 2500  |

---
**Query #2** <br>
What is the average unique products purchased in each transaction?
```sql
    WITH temp_table AS (
    SELECT COUNT(DISTINCT prod_id) as product_count
    FROM balanced_tree.sales
    GROUP BY txn_id
    )
    SELECT ROUND(AVG(product_count)) as unique_products
    FROM temp_table;
```
| unique_products |
| --------------- |
| 6               |

---
**Query #3** <br>
What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
    WITH revenue AS (
    SELECT SUM(price * qty) as revenue
    FROM balanced_tree.sales
    GROUP BY txn_id, discount
    )
    SELECT PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) AS percentile_25,
    	   PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS percentile_50,
           PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) AS percentile_75
    FROM revenue;
```
| percentile_25 | percentile_50 | percentile_75 |
| ------------- | ------------- | ------------- |
| 375.75        | 509.5         | 647           |

---
**Query #4** <br>
What is the average discount value per transaction?
```sql
    WITH discounts AS (
    SELECT txn_id,
    	   SUM(qty * price * discount :: numeric / 100) AS discount
    FROM balanced_tree.sales
    GROUP BY txn_id
    )
    SELECT ROUND(AVG(discount), 2) as avg_discount
    FROM discounts;
```
| avg_discount |
| ------------ |
| 62.49        |

---
**Query #5** <br>
What is the percentage split of all transactions for members vs non-members?
```sql
    SELECT ROUND(100 * (SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales WHERE member = 't')::NUMERIC / COUNT(DISTINCT txn_id)) AS member_percentage, 
    	   ROUND(100 * (SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales WHERE member = 'f')::NUMERIC / COUNT(DISTINCT txn_id)) AS non_member_percentage
    FROM balanced_tree.sales;
```
| member_percentage | non_member_percentage |
| ----------------- | --------------------- |
| 60                | 40                    |

---
**Query #6** <br>
What is the average revenue for member transactions and non-member transactions ?
```sql
    WITH txns AS (
    SELECT txn_id, 
    	   member,
           SUM(price * qty) as revenue
    FROM balanced_tree.sales
    GROUP BY txn_id, member
    )
    SELECT CASE WHEN member = 'true' THEN 'member' ELSE 'non-member' END AS status,
    	   ROUND(AVG(revenue)) as revenue
    FROM txns
    GROUP BY member;
```
| status     | revenue |
| ---------- | ------- |
| non-member | 515     |
| member     | 516     |
