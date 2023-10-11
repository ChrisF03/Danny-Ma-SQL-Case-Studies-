## D. Outside The Box Questions

The following are open ended questions which might be asked during a technical interview for this case study - there are no right or wrong answers, but answers that make sense from both a technical and a business perspective make an amazing impression!

**1. How would you calculate the rate of growth for Foodie-Fi?**
```sql
WITH subs AS (
SELECT DATE_TRUNC('month', start_date) AS month,
  		DATE_TRUNC('year', start_date) AS year,
	   COUNT(customer_id) AS number_of_subscribers,
       --past customers
       LAG(COUNT(customer_id), 1) over (
         ORDER BY
         DATE_TRUNC('month', start_date)) AS prev_number_of_subscribers,
         -- percentage growth
         (100 * (COUNT(customer_id) - LAG(COUNT(customer_id), 1) over ( 
           ORDER BY
          DATE_TRUNC('month', start_date))) / LAG(COUNT(customer_id),1) 
          over ( 
            ORDER BY 
            DATE_TRUNC('month', start_date))) || '%' AS Monthly_Percent_Change
FROM foodie_fi.subscriptions
WHERE plan_id <> 0 AND plan_id <> 4
GROUP BY month, year
ORDER BY month, year
)
SELECT CONCAT(TO_CHAR(month, 'Month'),'', EXTRACT(YEAR FROM year)) AS month,
	   number_of_subscribers,
       prev_number_of_subscribers,
       Monthly_Percent_Change
FROM subs;
```
| month         | number_of_subscribers | prev_number_of_subscribers | monthly_percent_change |
| ------------- | --------------------- | -------------------------- | ---------------------- |
| January  2020 | 62                    |                            |                        |
| February 2020 | 71                    | 62                         | 14%                    |
| March    2020 | 93                    | 71                         | 30%                    |
| April    2020 | 85                    | 93                         | -8%                    |
| May      2020 | 105                   | 85                         | 23%                    |
| June     2020 | 106                   | 105                        | 0%                     |
| July     2020 | 104                   | 106                        | -1%                    |
| August   2020 | 134                   | 104                        | 28%                    |
| September2020 | 115                   | 134                        | -14%                   |
| October  2020 | 125                   | 115                        | 8%                     |
| November 2020 | 101                   | 125                        | -19%                   |
| December 2020 | 111                   | 101                        | 9%                     |
| January  2021 | 58                    | 111                        | -47%                   |
| February 2021 | 29                    | 58                         | -50%                   |
| March    2021 | 24                    | 29                         | -17%                   |
| April    2021 | 20                    | 24                         | -16%                   |
---
**2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?**

- revenue (total, month/month, per user)
- expenses (total, month/month, per show license)
- overall number of subscribers (incl. trials)
- trial conversions (free:paying customers)
- subscribers per plan (gain insight into favorite features)
---
**3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?**

I would recommend closely tracking engagement on the app. Analyzing the frequency of use among subscribers as well as what kind of content they're most engaging with. This can help point towards potential focus areas for Foodie-Fi. 

Also, keeping track of how often the free trials are being converted to paying subscriptions and trying to gauge what could be improved on to better entice those subscribers.

---
**4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?**

- What's your reason for cancelling ? 
- What did you like about Foodie-Fi ?
- What did you dislike about Foodie-Fi ? 
- How can Foodie-Fi be improved ? 
- Overall, how would you rate Foodie-Fi ? (1-5 stars)
---
**5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?**

- Allow limited amount of access to Foodie-Fi for those who have completed their trial
- Offer first month discount 
- Leverage email/text marketing for targeted ad campaigns to remind subscribers about what Foodie-Fi has to offer as well as to bring attention to any new features.
- Offer perks to loyal, paying customers such as early access to new content, etc.; A loyalty program of some sort.
- Make it easy for subscribers to give feedback and/or bring up any issues they may be having on the platform. 

- These ideas can be validated through A/B testing, conversion rate, revenue generation and through cohort analysis to see if subscribers are staying on or jumping ship.
