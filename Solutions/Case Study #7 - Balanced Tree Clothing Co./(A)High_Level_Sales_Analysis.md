## A. High Level Sales Analysis

For this section, I queried for cumulative results as well as for each product.

**Query #1** <br>
What was the total quantity sold for all products?

- ALL PRODUCTS
```sql
    SELECT SUM(qty) as Total_Sold
    FROM balanced_tree.sales;
```
| total_sold |
| ---------- |
| 45216      |

- EACH PRODUCT
```sql
    SELECT t1.product_name,
    	   SUM(t2.qty) as Total_Sold
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2 ON
    t1.product_id = t2.prod_id
    GROUP BY t1.product_name;
```
| product_name                     | total_sold |
| -------------------------------- | ---------- |
| White Tee Shirt - Mens           | 3800       |
| Navy Solid Socks - Mens          | 3792       |
| Grey Fashion Jacket - Womens     | 3876       |
| Navy Oversized Jeans - Womens    | 3856       |
| Pink Fluro Polkadot Socks - Mens | 3770       |
| Khaki Suit Jacket - Womens       | 3752       |
| Black Straight Jeans - Womens    | 3786       |
| White Striped Socks - Mens       | 3655       |
| Blue Polo Shirt - Mens           | 3819       |
| Indigo Rain Jacket - Womens      | 3757       |
| Cream Relaxed Jeans - Womens     | 3707       |
| Teal Button Up Shirt - Mens      | 3646       |

---
**Query #2** <br>
What is the total generated revenue for all products before discounts?

- ALL PRODUCTS
```sql
    SELECT SUM(price*qty) as gross_revenue
    FROM balanced_tree.sales;
```
| gross_revenue |
| ------------- |
| 1289453       |


- EACH PRODUCT
```sql
    SELECT t1.product_name,
    	   SUM(t2.price * t2.qty) as gross_revenue
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2 ON
    t1.product_id = t2.prod_id
    GROUP BY t1.product_name;
```
| product_name                     | gross_revenue |
| -------------------------------- | ------------- |
| White Tee Shirt - Mens           | 152000        |
| Navy Solid Socks - Mens          | 136512        |
| Grey Fashion Jacket - Womens     | 209304        |
| Navy Oversized Jeans - Womens    | 50128         |
| Pink Fluro Polkadot Socks - Mens | 109330        |
| Khaki Suit Jacket - Womens       | 86296         |
| Black Straight Jeans - Womens    | 121152        |
| White Striped Socks - Mens       | 62135         |
| Blue Polo Shirt - Mens           | 217683        |
| Indigo Rain Jacket - Womens      | 71383         |
| Cream Relaxed Jeans - Womens     | 37070         |
| Teal Button Up Shirt - Mens      | 36460         |

---
**Query #3** <br>
What was the total discount amount for all products?

- ALL PRODUCTS
```sql
    SELECT ROUND(SUM((price * qty) * (discount::NUMERIC / 100)), 2) AS total_discounts
    FROM balanced_tree.sales;
```
| total_discounts |
| --------------- |
| 156229.14       |

- EACH PRODUCT
```sql
    SELECT t1.product_name,
    	   SUM(t2.price * t2.qty) as gross_revenue,
    	   ROUND(SUM((t2.price * t2.qty) * (t2.discount::NUMERIC / 100)), 2) AS total_discounts
    FROM balanced_tree.product_details as t1 
    JOIN balanced_tree.sales as t2 ON
    t1.product_id = t2.prod_id
    GROUP BY t1.product_name;
```
| product_name                     | gross_revenue | total_discounts |
| -------------------------------- | ------------- | --------------- |
| White Tee Shirt - Mens           | 152000        | 18377.60        |
| Navy Solid Socks - Mens          | 136512        | 16650.36        |
| Grey Fashion Jacket - Womens     | 209304        | 25391.88        |
| Navy Oversized Jeans - Womens    | 50128         | 6135.61         |
| Pink Fluro Polkadot Socks - Mens | 109330        | 12952.27        |
| Khaki Suit Jacket - Womens       | 86296         | 10243.05        |
| Black Straight Jeans - Womens    | 121152        | 14744.96        |
| White Striped Socks - Mens       | 62135         | 7410.81         |
| Blue Polo Shirt - Mens           | 217683        | 26819.07        |
| Indigo Rain Jacket - Womens      | 71383         | 8642.53         |
| Cream Relaxed Jeans - Womens     | 37070         | 4463.40         |
| Teal Button Up Shirt - Mens      | 36460         | 4397.60         |
