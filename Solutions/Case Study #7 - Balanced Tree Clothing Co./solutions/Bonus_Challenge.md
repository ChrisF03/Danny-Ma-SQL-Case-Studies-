## Bonus Challenge

Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.

```sql
SELECT t1.product_id,
       t1.price,
       CONCAT(t2.level_text, ' ', 
            CASE WHEN t2.parent_id = 3 THEN 'Jeans'
       		 WHEN t2.parent_id = 4 THEN 'Jacket'
            	 WHEN t2.parent_id = 5 THEN 'Shirt'
            	 WHEN t2.parent_id = 6 THEN 'Socks' END, 
              ' - ', 
            CASE WHEN t2.parent_id = 1 OR 
              	      t2.parent_id = 3 OR 
              	      t2.parent_id = 4
              	      THEN 'Womens' 
              	      ELSE 'Mens' END) as product_name,
       CASE WHEN t2.parent_id = 1 OR 
		 t2.parent_id = 3 OR 
                 t2.parent_id = 4 
                 THEN 1 
                 ELSE 2 END as category_id,
       CASE WHEN t2.parent_id = 3 THEN 3
       	    WHEN t2.parent_id = 4 THEN 4
            WHEN t2.parent_id = 5 THEN 5
            WHEN t2.parent_id = 6 THEN 6
            END AS segment_id,
       t1.id as style_id,
       CASE WHEN t2.parent_id = 1 OR 
       		 t2.parent_id = 3 OR 
                 t2.parent_id = 4 
                 THEN 'Womens'
                 ELSE 'Mens' END AS category_name,
       CASE WHEN t2.parent_id = 3 THEN 'Jeans'
       	    WHEN t2.parent_id = 4 THEN 'Jacket'
            WHEN t2.parent_id = 5 THEN 'Shirt'
            WHEN t2.parent_id = 6 THEN 'Socks'
            END AS segment_name,
       t2.level_text as style_name
FROM balanced_tree.product_prices as t1
JOIN balanced_tree.product_hierarchy as t2 ON
t1.id = t2.id 
GROUP BY t1.product_id, t1.price, t2.level_text, t2.parent_id, t2.id, t1.id;
```
| product_id | price | product_name                     | category_id | segment_id | style_id | category_name | segment_name | style_name          |
| ---------- | ----- | -------------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | ------------------- |
| 2a2353     | 57    | Blue Polo Shirt - Mens           | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo           |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens      | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain         |
| e83aa3     | 32    | Black Straight Jeans - Womens    | 1           | 3          | 8        | Womens        | Jeans        | Black Straight      |
| c4a632     | 13    | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| e31d39     | 10    | Cream Relaxed Jeans - Womens     | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed       |
| f084eb     | 36    | Navy Solid Socks - Mens          | 2           | 6          | 16       | Mens          | Socks        | Navy Solid          |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens       | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit          |
| b9a74d     | 17    | White Striped Socks - Mens       | 2           | 6          | 17       | Mens          | Socks        | White Striped       |
| 9ec847     | 54    | Grey Fashion Jacket - Womens     | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion        |
| c8d436     | 10    | Teal Button Up Shirt - Mens      | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up      |
| 5d267b     | 40    | White Tee Shirt - Mens           | 2           | 5          | 13       | Mens          | Shirt        | White Tee           |
| 2feb6b     | 29    | Pink Fluro Polkadot Socks - Mens | 2           | 6          | 18       | Mens          | Socks        | Pink Fluro Polkadot |
