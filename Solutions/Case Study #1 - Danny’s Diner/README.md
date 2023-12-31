<a href="https://8weeksqlchallenge.com/case-study-1/"> <img align="right" width="300" height="300" src="https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%231%20-%20Danny%E2%80%99s%20Diner/1.png"></a>

Danny Ma's 8-week SQL Challenge provides us multiple realistic end to end case studies that closely represent the sort of work performed in a data analytics role and includes datasets and case studies from the following domains:

* Health analytics
* Marketing analytics
* People analytics
* Financial markets
* Fast moving consumer goods
* Digital marketing

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!
<!-- Danny has shared with you 3 key datasets for this case study:

- sales
- menu
- members -->
This case study focused heavily on the following functions :<br> - Common Table Expressions<br> - Grouping By Aggregates<br> - Window Functions for ranking and<br> - Table Joins

## Entity Relationship Diagram
![Danny's Diner (1)](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/assets/103148784/6dc125a6-f3ea-4429-998f-1e4bd93ada5e)
---
<details>
<summary>
CASE STUDY
</summary>
    
## Case Study

**Query #1**<br>
What is the total amount each customer spent at the restaurant?
```sql
    SELECT sales.customer_id, 
    	   SUM(menu.price) as Total_Spent
    FROM dannys_diner.sales
    JOIN dannys_diner.menu ON
    sales.product_id = menu.product_id
    GROUP BY sales.customer_id
    ORDER by sales.customer_id;
```
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
**Query #2** <br>
How many days has each customer visited the restaurant?
```sql
    SELECT customer_id, 
    	   COUNT(DISTINCT order_date) as Total_Days
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY customer_id;
```
| customer_id | total_days |
| ----------- | ---------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

---
**Query #3**<br>
What was the first item from the menu purchased by each customer?
```sql
    WITH first_sales AS (
      SELECT sales.customer_id, 
          	 sales.order_date, 
          	 menu.product_name,
          	 DENSE_RANK() OVER (
               PARTITION BY sales.customer_id 
               ORDER BY sales.order_date) AS rank
      FROM dannys_diner.sales
      JOIN dannys_diner.menu ON 
      sales.product_id = menu.product_id
    )
    SELECT customer_id, 
    	   product_name
    FROM first_sales
    WHERE rank = 1
    GROUP BY customer_id, product_name;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---
**Query #4**<br>
What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
    SELECT menu.product_name,
    	   COUNT(sales.product_id) as Total_Purchased
    FROM dannys_diner.menu
    JOIN dannys_diner.sales ON
    menu.product_id = sales.product_id
    GROUP BY menu.product_name
    ORDER BY Total_Purchased DESC
    LIMIT 1;
```
| product_name | total_purchased |
| ------------ | --------------- |
| ramen        | 8               |

---
**Query #5**<br>
Which item was the most popular for each customer?
```sql
    With most_popular AS (
    SELECT sales.customer_id,
    	   menu.product_name,
           COUNT(sales.product_id) as Total_Purchased,
           DENSE_RANK() OVER (
             PARTITION BY sales.customer_id
             ORDER BY COUNT(sales.product_id) DESC) as rank
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu ON 
    sales.product_id = menu.product_id
    GROUP BY sales.customer_id, menu.product_name
    ORDER BY sales.customer_id
    )
    SELECT customer_id, 
    	   product_name,
           total_purchased
    FROM most_popular 
    WHERE rank = 1;
```
| customer_id | product_name | total_purchased |
| ----------- | ------------ | --------------- |
| A           | ramen        | 3               |
| B           | ramen        | 2               |
| B           | curry        | 2               |
| B           | sushi        | 2               |
| C           | ramen        | 3               |

---
**Query #6**<br>
Which item was purchased first by the customer after they became a member?
```sql
    With first_purchase AS (
    SELECT members.customer_id, 
           sales.product_id, 
           ROW_NUMBER() OVER(
             PARTITION BY members.customer_id 
             ORDER BY sales.order_date) AS row_
    FROM dannys_diner.members
    JOIN dannys_diner.sales ON
    members.customer_id = sales.customer_id AND 
    sales.order_date > members.join_date
    )
    SELECT customer_id,
    	   menu.product_name
    FROM first_purchase 
    JOIN dannys_diner.menu ON
    first_purchase.product_id = menu.product_id
    WHERE row_ = 1 
    GROUP BY customer_id, menu.product_name
    ORDER BY customer_id;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

---
**Query #7** <br>
Which item was purchased just before the customer became a member?
```sql
    WITH prior_purchase AS (
    SELECT members.customer_id, 
           sales.product_id, 
      	   DENSE_RANK() OVER (
             PARTITION BY members.customer_id 
             ORDER BY sales.order_date DESC) AS rank
    FROM dannys_diner.members
    JOIN dannys_diner.sales ON
    members.customer_id = sales.customer_id AND
    sales.order_date < members.join_date
    )
    SELECT prior_purchase.customer_id, 
           menu.product_name 
    FROM prior_purchase
    JOIN dannys_diner.menu ON
    prior_purchase.product_id = menu.product_id
    WHERE rank = 1
    ORDER BY prior_purchase.customer_id ASC;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

---
**Query #8** <br>
What is the total items and amount spent for each member before they became a member?
```sql
    SELECT sales.customer_id, 
    	   COUNT(sales.product_id) AS total_items, 
           SUM(menu.price) AS total_sales
    FROM dannys_diner.sales
    JOIN dannys_diner.members ON 
    sales.customer_id = members.customer_id AND
    sales.order_date < members.join_date
    JOIN dannys_diner.menu ON
    sales.product_id = menu.product_id
    GROUP BY sales.customer_id
    ORDER BY sales.customer_id;
```
| customer_id | total_items | total_sales |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

---
**Query #9** <br>
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
    WITH points_cte AS (
    SELECT menu.product_id, 
           CASE WHEN 
          		product_id = 1 THEN price * 20
              	ELSE price * 10
            	END AS points
    FROM dannys_diner.menu
    )
    SELECT sales.customer_id, 
           SUM(points_cte.points) AS total_points
    FROM dannys_diner.sales
    JOIN points_cte
    ON sales.product_id = points_cte.product_id
    GROUP BY sales.customer_id
    ORDER BY sales.customer_id;
```
| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---
**Query #10** <br>
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
    WITH dates_cte AS (
    SELECT customer_id, 
           join_date, 
           join_date + 6 AS valid_date, 
           DATE_TRUNC('month', '2021-01-31'::DATE)
              + interval '1 month' 
              - interval '1 day' AS last_date
    FROM dannys_diner.members
    )
    SELECT sales.customer_id, 
           SUM(CASE WHEN 
               menu.product_name = 'sushi' THEN 2 * 10 * menu.price
               WHEN sales.order_date BETWEEN 
               dates.join_date AND dates.valid_date 
               THEN 2 * 10 * menu.price
               ELSE 10 * menu.price END) AS points
    FROM dannys_diner.sales
    JOIN dates_cte AS dates ON
    sales.customer_id = dates.customer_id AND
    sales.order_date <= dates.last_date
    JOIN dannys_diner.menu ON
    sales.product_id = menu.product_id
    GROUP BY sales.customer_id;
```
| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |
</details>

<details>
<summary>BONUS QUESTIONS</summary>
    
## JOIN ALL THE THINGS
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:
| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01               | curry        | 15    | N      |
| A           | 2021-01-01               | sushi        | 10    | N      |
| A           | 2021-01-07               | curry        | 15    | Y      |
| A           | 2021-01-10               | ramen        | 12    | Y      |
| A           | 2021-01-11               | ramen        | 12    | Y      |
| A           | 2021-01-11               | ramen        | 12    | Y      |
| B           | 2021-01-01               | curry        | 15    | N      |
| B           | 2021-01-02               | curry        | 15    | N      |
| B           | 2021-01-04               | sushi        | 10    | N      |
| B           | 2021-01-11               | sushi        | 10    | Y      |
| B           | 2021-01-16               | ramen        | 12    | Y      |
| B           | 2021-02-01               | ramen        | 12    | Y      |
| C           | 2021-01-01               | ramen        | 12    | N      |
| C           | 2021-01-01               | ramen        | 12    | N      |
| C           | 2021-01-07               | ramen        | 12    | N      |
```sql
    SELECT sales.customer_id, 
    	   sales.order_date, 
           menu.product_name,
           menu.price,
           CASE 
           WHEN sales.order_date < members.join_date THEN 'N'
           WHEN sales.order_date >= members.join_date THEN 'Y'
           ELSE 'N'
           END AS member
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu ON
    sales.product_id = menu.product_id
    LEFT JOIN dannys_diner.members ON 
    sales.customer_id = members.customer_id
    ORDER BY members.customer_id, sales.order_date, menu.product_name;
```
| customer_id | order_date               | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01               | curry        | 15    | N      |
| A           | 2021-01-01               | sushi        | 10    | N      |
| A           | 2021-01-07               | curry        | 15    | Y      |
| A           | 2021-01-10               | ramen        | 12    | Y      |
| A           | 2021-01-11               | ramen        | 12    | Y      |
| A           | 2021-01-11               | ramen        | 12    | Y      |
| B           | 2021-01-01               | curry        | 15    | N      |
| B           | 2021-01-02               | curry        | 15    | N      |
| B           | 2021-01-04               | sushi        | 10    | N      |
| B           | 2021-01-11               | sushi        | 10    | Y      |
| B           | 2021-01-16               | ramen        | 12    | Y      |
| B           | 2021-02-01               | ramen        | 12    | Y      |
| C           | 2021-01-01               | ramen        | 12    | N      |
| C           | 2021-01-01               | ramen        | 12    | N      |
| C           | 2021-01-07               | ramen        | 12    | N      |
<br>

## RANK ALL THE THINGS
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

Recreate the following table output using the available data:
| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01               | curry        | 15    | N      | NULL    |
| A           | 2021-01-01               | sushi        | 10    | N      | NULL    |
| A           | 2021-01-07               | curry        | 15    | Y      | 1       |
| A           | 2021-01-10               | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11               | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11               | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01               | curry        | 15    | N      | NULL    |
| B           | 2021-01-02               | curry        | 15    | N      | NULL    |
| B           | 2021-01-04               | sushi        | 10    | N      | NULL    |
| B           | 2021-01-11               | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16               | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01               | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01               | ramen        | 12    | N      | NULL    |
| C           | 2021-01-01               | ramen        | 12    | N      | NULL    |
| C           | 2021-01-07               | ramen        | 12    | N      | NULL    |


```sql
    With customers AS (
    SELECT sales.customer_id, 
    	   sales.order_date, 
           menu.product_name,
           menu.price,
           CASE 
           WHEN sales.order_date < members.join_date THEN 'N'
           WHEN sales.order_date >= members.join_date THEN 'Y'
           ELSE 'N'
           END AS member
    FROM dannys_diner.sales 
    JOIN dannys_diner.menu ON
    sales.product_id = menu.product_id
    LEFT JOIN dannys_diner.members ON 
    sales.customer_id = members.customer_id
    ORDER BY members.customer_id, sales.order_date, menu.product_name
    ) 
    SELECT *, 
    	   CASE
           WHEN member = 'N' THEN NULL
           ELSE RANK () OVER(
          PARTITION BY customer_id, member
          ORDER BY order_date) END AS ranking
    FROM customers;
```
| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01               | curry        | 15    | N      | NULL    |
| A           | 2021-01-01               | sushi        | 10    | N      | NULL    |
| A           | 2021-01-07               | curry        | 15    | Y      | 1       |
| A           | 2021-01-10               | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11               | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11               | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01               | curry        | 15    | N      | NULL    |
| B           | 2021-01-02               | curry        | 15    | N      | NULL    |
| B           | 2021-01-04               | sushi        | 10    | N      | NULL    |
| B           | 2021-01-11               | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16               | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01               | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01               | ramen        | 12    | N      | NULL    |
| C           | 2021-01-01               | ramen        | 12    | N      | NULL    |
| C           | 2021-01-07               | ramen        | 12    | N      | NULL    |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
</details>
