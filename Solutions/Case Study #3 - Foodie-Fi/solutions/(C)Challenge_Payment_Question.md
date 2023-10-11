## C. Challenge Payment Question

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

```diff
- INCOMPLETE
```

```sql
-- SELECT t1.customer_id,
-- 	      t1.plan_id,
--        t2.plan_name,
--        t1.start_date as payment_date,
--        t2.price,
--        ROW_NUMBER() OVER (
--          PARTITION BY customer_id) AS Payment_Order
-- FROM foodie_fi.subscriptions AS t1 
-- JOIN foodie_fi.plans AS t2 ON
-- t1.plan_id = t2.plan_id 
-- WHERE t1.plan_id != 0
-- GROUP BY t1.customer_id, t1.plan_id, t2.plan_name, t1.start_date, t2.price
-- ORDER BY t1.customer_id;
```
