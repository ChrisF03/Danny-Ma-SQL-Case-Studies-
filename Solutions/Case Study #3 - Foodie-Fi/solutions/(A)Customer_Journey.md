## A. Customer Journey

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```sql
SELECT t1.customer_id,
	   t2.plan_name,
       t1.start_date
FROM foodie_fi.subscriptions as t1
JOIN foodie_fi.plans as t2 ON
t1.plan_id = t2.plan_id
WHERE t1.customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY t1.customer_id, t1.start_date;
```
| customer_id | plan_name     | start_date |
| ----------- | ------------- | -----------|
| 1           | trial         | 2020-08-01 |
| 1           | basic monthly | 2020-08-08 |
| 2           | trial         | 2020-09-20 |
| 2           | pro annual    | 2020-09-27 |
| 11          | trial         | 2020-11-19 |
| 11          | churn         | 2020-11-26 |
| 13          | trial         | 2020-12-15 |
| 13          | basic monthly | 2020-12-22 |
| 13          | pro monthly   | 2021-03-29 |
| 15          | trial         | 2020-03-17 |
| 15          | pro monthly   | 2020-03-24 |
| 15          | churn         | 2020-04-29 |
| 16          | trial         | 2020-05-31 |
| 16          | basic monthly | 2020-06-07 |
| 16          | pro annual    | 2020-10-21 |
| 18          | trial         | 2020-07-06 |
| 18          | pro monthly   | 2020-07-13 |
| 19          | trial         | 2020-06-22 |
| 19          | pro monthly   | 2020-06-29 |
| 19          | pro annual    | 2020-08-29 |

- **Client #1**: upgraded to a basic monthly plan at the end of their free trial
- **Client #2**: upgraded to a pro annual plan at the end of their free trial
- **Client #11**: cancelled their subscription at the end of the trial
- **Client #13**: upgraded to the basic monthly plan at the end of their trial and to the pro monthly plan a little over 3 months after that.
- **Client #15**: upgraded to a pro monthly plan at the end of their trial and then churned a little over a month after that.
- **Client #16**: upgraded to the basic monthly plan at the end of their trial and to the pro annual plan nearly 5 months after that.
- **Client #18**: upgraded to the pro monthly plan at the end of their trial
- **Client #19**: upgraded to the pro monthly plan at the end of their trial and to the pro annual plan exactly 2 months after.
