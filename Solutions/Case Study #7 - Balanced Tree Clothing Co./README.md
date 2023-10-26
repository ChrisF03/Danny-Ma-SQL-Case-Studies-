<a href="https://8weeksqlchallenge.com/case-study-7/"> <img align="right" width="300" height="300" src="https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./7.png"></a>

Danny Ma's 8-week SQL Challenge provides us multiple realistic end to end case studies that closely represent the sort of work performed in a data analytics role and includes datasets and case studies from the following domains:

* Health analytics
* Marketing analytics
* People analytics
* Financial markets
* Fast moving consumer goods
* Digital marketing

## Introduction

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

Sales, transactions and product exposure is always going to be a main objective for many data analysts and data scientists when working within a company that sells some type of product - **Spoiler alert**: nearly all companies will sell products!

Being able to navigate your way around a product hierarchy and understand the different levels of the structures as well as being able to join these details to sales related datasets will be super valuable for anyone wanting to work within a financial, customer or exploratory analytics capacity.

## Data

For this case study there is a total of 4 datasets for this case study - however we will only need to utilise the 2 first tables to solve all of the regular questions, and the additional 2 tables are used only for the bonus challenge question!

<details>
<summary>balanced_tree.product_details</summary>
    
**ONLY TOP 5 RECORDS ARE SHOWN**
  
| product_id | price | product_name                  | category_id | segment_id | style_id | category_name | segment_name | style_name     |
| ---------- | ----- | ----------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | -------------- |
| c4a632     | 13    | Navy Oversized Jeans - Womens | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized |
| e83aa3     | 32    | Black Straight Jeans - Womens | 1           | 3          | 8        | Womens        | Jeans        | Black Straight |
| e31d39     | 10    | Cream Relaxed Jeans - Womens  | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed  |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens    | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit     |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens   | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain    |
</details>

<details>
<summary>balanced_tree.sales</summary>
  
**ONLY TOP 5 RECORDS ARE SHOWN**
  
| prod_id | qty | price | discount | member | txn_id | start_txn_time           |
| ------- | --- | ----- | -------- | ------ | ------ | ------------------------ |
| c4a632  | 4   | 13    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 5d267b  | 4   | 40    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| b9a74d  | 4   | 17    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| 2feb6b  | 2   | 29    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z |
| c4a632  | 5   | 13    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z |
</details>

<details>
<summary>balanced_tree.product_hierarchy</summary>
  
| id  | parent_id | level_text          | level_name |
| --- | --------- | ------------------- | ---------- |
| 1   | NULL      | Womens              | Category   |
| 2   | NULL      | Mens                | Category   |
| 3   | 1         | Jeans               | Segment    |
| 4   | 1         | Jacket              | Segment    |
| 5   | 2         | Shirt               | Segment    |
| 6   | 2         | Socks               | Segment    |
| 7   | 3         | Navy Oversized      | Style      |
| 8   | 3         | Black Straight      | Style      |
| 9   | 3         | Cream Relaxed       | Style      |
| 10  | 4         | Khaki Suit          | Style      |
| 11  | 4         | Indigo Rain         | Style      |
| 12  | 4         | Grey Fashion        | Style      |
| 13  | 5         | White Tee           | Style      |
| 14  | 5         | Teal Button Up      | Style      |
| 15  | 5         | Blue Polo           | Style      |
| 16  | 6         | Navy Solid          | Style      |
| 17  | 6         | White Striped       | Style      |
| 18  | 6         | Pink Fluro Polkadot | Style      |
</details>

<details>
<summary>balanced_tree.prices</summary>
  
| id  | product_id | price |
| --- | ---------- | ----- |
| 7   | c4a632     | 13    |
| 8   | e83aa3     | 32    |
| 9   | e31d39     | 10    |
| 10  | d5e9a6     | 23    |
| 11  | 72f5d4     | 19    |
| 12  | 9ec847     | 54    |
| 13  | 5d267b     | 40    |
| 14  | c8d436     | 10    |
| 15  | 2a2353     | 57    |
| 16  | f084eb     | 36    |
| 17  | b9a74d     | 17    |
| 18  | 2feb6b     | 29    |
</details>

## Case Study
A. [High Level Sales Analysis](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./solutions/(A)High_Level_Sales_Analysis.md) <br>
B. [Transaction Analysis](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./solutions/(B)Transaction_Analysis.md) <br>
C. [Product Analysis](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./solutions/(C)Product_Analysis.md) <br>
D. [Reporting Challenge](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./solutions/(D)Reporting_Challenge.md) <br>
E. [Bonus Challenge](https://github.com/ChrisF03/Danny-Ma-SQL-Case-Studies-/blob/main/Solutions/Case%20Study%20%237%20-%20Balanced%20Tree%20Clothing%20Co./solutions/Bonus_Challenge.md)
