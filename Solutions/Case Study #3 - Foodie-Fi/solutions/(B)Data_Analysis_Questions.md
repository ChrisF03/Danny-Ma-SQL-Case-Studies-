## B. Data Analysis Questions

**Query #1** <br>
How many customers has Foodie-Fi ever had?
```sql
    SELECT COUNT(DISTINCT customer_id) as total_customers
    FROM foodie_fi.subscriptions;
```
| total_customers |
| --------------- |
| 1000            |

---
**Query #2** <br>
What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.
```sql
    WITH subs AS(
    SELECT DATE_TRUNC('month', start_date)::DATE AS start_of_month,
    	   COUNT(*) as total_trials
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    GROUP BY start_of_month
    ORDER BY start_of_month
    )
    SELECT TO_CHAR(start_of_month, 'Month') as Month, 
    	   total_trials 
    FROM subs;
```
| month     | total_trials |
| --------- | ------------ |
| January   | 88           |
| February  | 68           |
| March     | 94           |
| April     | 81           |
| May       | 88           |
| June      | 79           |
| July      | 89           |
| August    | 88           |
| September | 87           |
| October   | 79           |
| November  | 75           |
| December  | 84           |

---
**Query #3** <br>
What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.
```sql
    SELECT t1.plan_id,
    	   t1.plan_name,
           COUNT(t2.start_date) as subscriptions
    FROM foodie_fi.plans as t1
    JOIN foodie_fi.subscriptions as t2 ON
    t1.plan_id = t2.plan_id
    WHERE t2.start_date >= '2021/01/01'
    GROUP BY t1.plan_id, t1.plan_name
    ORDER BY t1.plan_id;
```
| plan_id | plan_name     | subscriptions |
| ------- | ------------- | ------------- |
| 1       | basic monthly | 8             |
| 2       | pro monthly   | 60            |
| 3       | pro annual    | 63            |
| 4       | churn         | 71            |

---
**Query #4** <br>
What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
    with total AS (
    SELECT COUNT(DISTINCT customer_id) as total_customers
    FROM foodie_fi.subscriptions
    ),
    churn AS (
    SELECT COUNT(customer_id) as amount_churned
    FROM foodie_fi.subscriptions 
    WHERE plan_id = 4
    )
    SELECT total.total_customers,
    	   churn.amount_churned,
           ROUND((amount_churned::NUMERIC/total_customers)*100, 1) as percentage
    FROM total, churn;
```
| total_customers | amount_churned | percentage |
| --------------- | -------------- | ---------- |
| 1000            | 307            | 30.7       |

---
**Query #5** <br>
How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
    WITH rows AS (
    SELECT DISTINCT t2.customer_id,
      	   t1.plan_name,
      	   t2.plan_id,
      	   ROW_NUMBER() OVER (PARTITION BY t2.customer_id ORDER BY t2.plan_id) AS rn
    FROM foodie_fi.subscriptions as t2
    JOIN foodie_fi.plans as t1 
    ON t1.plan_id = t2.plan_id
    ORDER BY t2.customer_id, t2.plan_id
    ),
    get_rows AS (
    SELECT customer_id,
    	   plan_name,
           rn
    FROM rows
    WHERE rn < 3
    ),
    get_churn AS (
    SELECT SUM(CASE
               	   WHEN rn = 2 AND plan_name = 'churn' THEN 1
                   ELSE 0
                   END) AS trial_users_only
    FROM get_rows
    )
    SELECT COUNT(DISTINCT t1.customer_id) AS total_customers ,
    	   t2.* AS trial_users_only,
           ROUND((t2.trial_users_only::NUMERIC / COUNT(DISTINCT t1.customer_id)) * 100, 1) AS trial_churn_percentage
    FROM rows as t1, get_churn as t2
    GROUP BY trial_users_only;
```
| total_customers | trial_users_only | trial_churn_percentage |
| --------------- | ---------------- | ---------------------- |
| 1000            | 92               | 9.2                    |

---
**Query #6** <br>
What is the number and percentage of customer plans after their initial free trial?
```sql
    with rows AS(
    SELECT DISTINCT t2.customer_id as Total,
      	   t1.plan_name,
      	   t2.plan_id,
      	   ROW_NUMBER() OVER (PARTITION BY t2.customer_id ORDER BY t2.plan_id) AS rn
    FROM foodie_fi.subscriptions as t2
    JOIN foodie_fi.plans as t1 
    ON t1.plan_id = t2.plan_id
    ORDER BY t2.customer_id, t2.plan_id
    ),
    total_customers AS(
    SELECT COUNT(DISTINCT customer_id) as total_customers
    FROM foodie_fi.subscriptions
    )
    SELECT plan_name,
    	   COUNT(plan_name) as Total_Subs,
           ROUND((COUNT(plan_name)::NUMERIC / total_customers) *100,1) AS plan_percentage
    FROM rows, total_customers
    WHERE rn = 2
    GROUP BY plan_id, plan_name, total_customers
    ORDER BY plan_id;
```
| plan_name     | total_subs | plan_percentage |
| ------------- | ---------- | --------------- |
| basic monthly | 546        | 54.6            |
| pro monthly   | 325        | 32.5            |
| pro annual    | 37         | 3.7             |
| churn         | 92         | 9.2             |

---
**Query #7** <br>
What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql
    WITH next_sub AS (
    SELECT t1.customer_id,
      	   t1.plan_id,
           t2.plan_name,
      	   t1.start_date,
      	   LEAD(t1.start_date) OVER (
             PARTITION BY t1.customer_id
             ORDER BY t1.start_date
        		) AS next_sub
    FROM foodie_fi.subscriptions as t1
    JOIN foodie_fi.plans as t2 ON
    t1.plan_id = t2.plan_id
    WHERE start_date <= '2020-12-31'
    ORDER BY customer_id, plan_id ASC
    ),
    total_customers AS(
    SELECT COUNT(DISTINCT customer_id) as total_customers
    FROM foodie_fi.subscriptions
    )
    SELECT t1.plan_name, 
    	   COUNT(DISTINCT t1.customer_id) as total_subs,
           ROUND((COUNT(plan_name)::NUMERIC / t2.total_customers) * 100, 1) as percent_of_total
    FROM next_sub as t1, total_customers as t2
    WHERE next_sub IS NULL
    GROUP BY t1.plan_name, t1.plan_id, t2.total_customers
    ORDER BY t1.plan_id ASC;
```
| plan_name     | total_subs | percent_of_total |
| ------------- | ---------- | ---------------- |
| trial         | 19         | 1.9              |
| basic monthly | 224        | 22.4             |
| pro monthly   | 326        | 32.6             |
| pro annual    | 195        | 19.5             |
| churn         | 236        | 23.6             |

---
**Query #8** <br>
How many customers have upgraded to an annual plan in 2020?
```sql
    SELECT COUNT(customer_id)
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3 AND
    start_date <= '2020-12-31';
```
| count |
| ----- |
| 195   |

---
**Query #9** <br>
How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
```sql
    WITH trial_plan AS (
    SELECT customer_id, 
      	   start_date AS trial_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    ), 
    annual_plan AS (
    SELECT customer_id, 
           start_date AS annual_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3
    )
    SELECT ROUND(AVG(annual_plan.annual_date - trial_plan.trial_date),0) AS avg_days_to_upgrade
    FROM trial_plan 
    JOIN annual_plan ON 
    trial_plan.customer_id = annual_plan.customer_id;
```
| avg_days_to_upgrade |
| ------------------- |
| 105                 |

---
**Query #10** <br>
Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
    WITH trial_plan AS (
    SELECT customer_id, 
      	   start_date AS trial_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    ), 
    annual_plan AS (
    SELECT customer_id, 
           start_date AS annual_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3
    ),
    date_periods AS (
    SELECT t1.customer_id,
    	   t1.trial_date,
           t2.annual_date,
           ((t2.annual_date - t1.trial_date) / 30 + 1) AS date_period
    FROM trial_plan as t1
    JOIN annual_plan as t2 ON
    t1.customer_id = t2.customer_id
    GROUP BY t1.customer_id, t1.trial_date, t2.annual_date
    )
    SELECT CASE
               WHEN date_period = 1 THEN CONCAT((date_period - 1),' - ', (date_period * 30),' days') 
               ELSE CONCAT(((date_period - 1) *30 + 1), ' - ', (date_period * 30), ' days')
               END as time_frame,
           COUNT(customer_id) AS Total_Customers,
           ROUND(AVG(annual_date - trial_date), 2) AS avg_days
    FROM date_periods
    GROUP BY time_frame
    ORDER BY time_frame;
```
| time_frame     | total_customers | avg_days |
| -------------- | --------------- | -------- |
| 0 - 30 days    | 48              | 9.54     |
| 31 - 60 days   | 25              | 41.84    |
| 61 - 90 days   | 33              | 70.88    |
| 91 - 120 days  | 35              | 99.83    |
| 121 - 150 days | 43              | 133.05   |
| 151 - 180 days | 35              | 161.54   |
| 181 - 210 days | 27              | 190.33   |
| 211 - 240 days | 4               | 224.25   |
| 241 - 270 days | 5               | 257.20   |
| 271 - 300 days | 1               | 285.00   |
| 301 - 330 days | 1               | 327.00   |
| 331 - 360 days | 1               | 346.00   |

---
**Query #11** <br>
How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
    WITH ranked_cte AS (
    SELECT sub.customer_id,  
      	   plans.plan_id,
           plans.plan_name,
           LEAD(plans.plan_id) OVER (
              PARTITION BY sub.customer_id
              ORDER BY sub.start_date) AS next_plan_id
      FROM foodie_fi.subscriptions AS sub
      JOIN foodie_fi.plans 
        ON sub.plan_id = plans.plan_id
     WHERE DATE_PART('year', start_date) = 2020
    )
      
    SELECT COUNT(customer_id) AS downgrading_customers
    FROM ranked_cte
    WHERE plan_id = 2
    AND next_plan_id = 1;
```
| downgrading_customers |
| --------------------- |
| 0                     |
