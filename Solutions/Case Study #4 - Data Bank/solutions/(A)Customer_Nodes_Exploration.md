## A. Customer Nodes Exploration

**Query #1** <br>
How many unique nodes are there on the Data Bank system?
```sql
    SELECT COUNT(DISTINCT node_id)
    FROM data_bank.customer_nodes;
```
| count |
| ----- |
| 5     |

---
**Query #2** <br>
What is the number of nodes per region?
```sql
    SELECT t1.region_name,
    	   COUNT(DISTINCT t2.node_id) as total_nodes
    FROM data_bank.regions as t1
    JOIN data_bank.customer_nodes as t2 ON
    t1.region_id = t2.region_id
    GROUP BY t1.region_name;
```
| region_name | total_nodes |
| ----------- | ----------- |
| Africa      | 5           |
| America     | 5           |
| Asia        | 5           |
| Australia   | 5           |
| Europe      | 5           |

---
**Query #3** <br>
How many customers are allocated to each region?
```sql
    SELECT t1.region_name,
    	   COUNT(DISTINCT t2.customer_id) as total_customers
    FROM data_bank.regions as t1
    JOIN data_bank.customer_nodes as t2 ON
    t1.region_id = t2.region_id
    GROUP BY t1.region_name;
```
| region_name | total_customers |
| ----------- | --------------- |
| Africa      | 102             |
| America     | 105             |
| Asia        | 95              |
| Australia   | 110             |
| Europe      | 88              |

---
**Query #4** <br>
How many days on average are customers reallocated to a different node?
```sql
    SELECT FLOOR(AVG(end_date::date - start_date::date)) as avg_days
    FROM data_bank.customer_nodes
    WHERE EXTRACT(YEAR FROM end_date) != '9999';
```
| avg_days |
| -------- |
| 14       |

---
**Query #5** <br>
What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
    WITH days_per_region AS (
    SELECT t1.region_name,
    	   (t2.end_date::date - t2.start_date::date) as days
    FROM data_bank.regions as t1
    JOIN data_bank.customer_nodes as t2 ON
    t1.region_id = t2.region_id
    WHERE EXTRACT(YEAR FROM end_date) != '9999'
    GROUP BY t1.region_name, t2.end_date, t2.start_date
    )
    SELECT region_name, 
    	   PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY days) AS median,
           PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY days) AS percentile_80,
           PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY days) AS percentile_95
    FROM days_per_region
    GROUP BY region_name;
```
| region_name | median | percentile_80 | percentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 15     | 24            | 28            |
| America     | 15     | 23            | 28            |
| Asia        | 14     | 23            | 28            |
| Australia   | 15     | 23            | 28            |
| Europe      | 15     | 24            | 28            |
