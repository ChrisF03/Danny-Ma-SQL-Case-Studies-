## 4. Campaigns Analysis

Generate a table that has 1 single row for every unique visit_id record and has the following columns:

`user_id` <br>
`visit_id` <br>
`visit_start_time`: the earliest event_time for each visit <br>
`page_views`: count of page views for each visit <br>
`cart_adds`: count of product cart add events for each visit <br>
`purchase`: 1/0 flag if a purchase event exists for each visit <br>
`campaign_name`: map the visit to a campaign if the visit_start_time falls between the start_date and end_date <br>
`impression`: count of ad impressions for each visit <br>
`click`: count of ad clicks for each visit <br>
(Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

```sql
CREATE TEMP TABLE campaign_analysis AS (
SELECT t1.user_id,
       t2.visit_id, 
       MIN(t2.event_time) as visit_start_time,
       SUM(CASE WHEN t2.event_type = 1 THEN 1 ELSE 0 END) as page_views,
       SUM(CASE WHEN t2.event_type = 2 THEN 1 ELSE 0 END) as cart_adds,
       CASE WHEN t2.event_type = 3 THEN 1 ELSE 0 END as purchase,
       t3.campaign_name,
       SUM(CASE WHEN t2.event_type = 4 THEN 1 ELSE 0 END) as impression,
       SUM(CASE WHEN t2.event_type = 5 THEN 1 ELSE 0 END) as click,
       STRING_AGG(CASE WHEN t4.product_id IS NOT NULL AND t2.event_type = 2 THEN t4.page_name ELSE NULL END, 
    ', ' ORDER BY t2.sequence_number) AS cart_products
FROM clique_bait.users as t1 
JOIN clique_bait.events as t2 ON
t1.cookie_id = t2.cookie_id
JOIN clique_bait.campaign_identifier as t3 ON
t2.event_time BETWEEN t3.start_date AND t3.end_date
LEFT JOIN clique_bait.page_hierarchy as t4 ON
t4.page_id = t2.page_id
GROUP BY t1.user_id, t2.visit_id, t2.event_type, t3.campaign_name
);
```
```sql
SELECT *
FROM campaign_analyis
LIMIT 10
```
| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchase | campaign_name                     | impression | click | cart_products                                                  |
| ------- | -------- | ------------------------ | ---------- | --------- | -------- | --------------------------------- | ---------- | ----- | -------------------------------------------------------------- |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1       | 0fc437   | 2020-02-04T17:52:04.205Z | 0          | 6         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster     |
| 1       | 0fc437   | 2020-02-04T17:59:31.521Z | 0          | 0         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1       | 0fc437   | 2020-02-04T17:49:51.435Z | 0          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 1          | 0     |                                                                |
| 1       | 0fc437   | 2020-02-04T17:50:10.091Z | 0          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 1     |                                                                |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
| 1       | 30b94d   | 2020-03-15T13:14:45.432Z | 0          | 7         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab |
| 1       | 30b94d   | 2020-03-15T13:21:11.621Z | 0          | 0         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                |
