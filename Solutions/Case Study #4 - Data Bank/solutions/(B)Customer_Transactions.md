## B. Customer Transactions

**Query #1** <br>
What is the unique count and total amount for each transaction type?
```sql
    SELECT DISTINCT txn_type,
    	   COUNT(*) as txn_count,
           SUM(txn_amount) as total_amount
    FROM data_bank.customer_transactions
    GROUP BY txn_type;
```
| txn_type   | txn_count | total_amount |
| ---------- | --------- | ------------ |
| withdrawal | 1580      | 793003       |
| purchase   | 1617      | 806537       |
| deposit    | 2671      | 1359168      |

---
**Query #2** <br>
What is the average total historical deposit counts and amounts for all customers?
```sql
    WITH deposits AS (
    SELECT customer_id, 
    	   COUNT(*) as total_count,
    	   AVG(txn_amount) as total_amount
    FROM data_bank.customer_transactions
    WHERE txn_type = 'deposit'
    GROUP BY customer_id
    )
    SELECT ROUND(AVG(total_count)) as total_deposit_count,
    	   ROUND(AVG(total_amount)) as total_deposit_amount
    FROM deposits;
```
| total_deposit_count | total_deposit_amount |
| ------------------- | -------------------- |
| 5                   | 509                  |

---
**Query #3** <br>
For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
    WITH monthly_transactions AS (
    SELECT DISTINCT customer_id,
      	   DATE_PART('Month', txn_date) as Month,
    	   SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
           SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
           SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
    FROM data_bank.customer_transactions
    GROUP BY customer_id, Month
    ORDER BY customer_id, Month
    )
    SELECT Month,
    	   COUNT(customer_id)
    FROM monthly_transactions
    WHERE deposit_count > 1 AND (withdrawal_count = 1 or purchase_count = 1)
    GROUP BY Month;
```
| month | count |
| ----- | ----- |
| 1     | 115   |
| 2     | 108   |
| 3     | 113   |
| 4     | 50    |

---
**Query #4** <br>
What is the closing balance for each customer at the end of the month?
```sql
WITH bal AS (
SELECT customer_id,
       DATE_TRUNC('Month', txn_date)::date AS txn_month,
       SUM(CASE WHEN txn_type='deposit' THEN txn_amount 
           ELSE -txn_amount 
           END) AS balance
FROM data_bank.customer_transactions
GROUP BY customer_id, txn_month
ORDER BY customer_id
)
SELECT *,
       SUM(balance) OVER (PARTITION BY customer_id
                          ORDER BY txn_month 
                          ) as Closing_balance
FROM bal
GROUP BY customer_id, txn_month, balance
ORDER BY customer_id;
```
* **only results for customer_id #1-3 are shown**

| customer_id | txn_month                | balance | closing_balance |
| ----------- | ------------------------ | ------- | --------------- |
| 1           | 2020-01-01T00:00:00.000Z | 312     | 312             |
| 1           | 2020-03-01T00:00:00.000Z | -952    | -640            |
| 2           | 2020-01-01T00:00:00.000Z | 549     | 549             |
| 2           | 2020-03-01T00:00:00.000Z | 61      | 610             |
| 3           | 2020-01-01T00:00:00.000Z | 144     | 144             |
| 3           | 2020-02-01T00:00:00.000Z | -965    | -821            |
| 3           | 2020-03-01T00:00:00.000Z | -401    | -1222           |
| 3           | 2020-04-01T00:00:00.000Z | 493     | -729            |
| ...         | ...                      | ...     | ...             |
| ...         | ...                      | ...     | ...             |

---
**Query #5** <br>
What is the percentage of customers who increase their closing balance by more than 5%?
```diff
- INCOMPLETE
```
