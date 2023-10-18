## 2. Data Exploration

**Cleaned Data Table**

```sql
    CREATE TEMP TABLE clean_weekly_sales AS (
    SELECT TO_DATE(week_date, 'dd/mm/yy') AS week_date,
    	   DATE_PART('week', TO_DATE(week_date, 'dd/mm/yy'))::INT AS week_number,
           DATE_PART('month', TO_DATE(week_date, 'dd/mm/yy'))::INT AS month_number,
           DATE_PART('year', TO_DATE(week_date, 'dd/mm/yy'))::INT AS calendar_year,
           region,
           platform,
           CASE 
           		WHEN segment = NULL OR segment = 'null' THEN 'unknown'
           		ELSE segment 
           END AS segment,
           CASE 
           		WHEN SUBSTRING(segment, 2, 1) = '1' THEN 'Young Adults'
      			WHEN SUBSTRING(segment, 2, 1) = '2' THEN 'Middle Aged'
      			WHEN SUBSTRING(segment, 2, 1) = '3' OR SUBSTRING(segment, 2, 1) = '4'  THEN 'Retirees'
    			ELSE 'unknown'
      	   END AS age_band,
      	   CASE 
      			WHEN SUBSTRING(segment, 1, 1) = 'C' THEN 'Couples'
      			WHEN SUBSTRING(segment, 1, 1) = 'F' THEN 'Families'
      			ELSE 'unknown'
      	   END AS demographic,
      	   customer_type,
           transactions,
           sales,
           ROUND((sales/transactions), 2) AS avg_transaction
    FROM data_mart.weekly_sales
    );
```

---
**Query #1** <br>
What day of the week is used for each week_date value?
```sql
    SELECT DISTINCT(TO_CHAR(week_date, 'day')) AS week_day 
    FROM clean_weekly_sales;
```
| week_day  |
| --------- |
| monday    |

---
**Query #2** <br>
What range of week numbers are missing from the dataset?
```sql
    WITH series AS (
    SELECT * 
    FROM generate_series(1,52,1)
    )
    SELECT t1.*
    FROM series as t1 
    LEFT JOIN clean_weekly_sales as t2 ON
    t1.generate_series = t2.week_number
    WHERE t2.week_number IS NULL
    GROUP BY t1.generate_series
    ORDER BY t1.generate_series;
```
| generate_series |
| --------------- |
| 1               |
| 2               |
| 3               |
| 4               |
| 5               |
| 6               |
| 7               |
| 8               |
| 9               |
| 10              |
| 11              |
| 12              |
| 37              |
| 38              |
| 39              |
| 40              |
| 41              |
| 42              |
| 43              |
| 44              |
| 45              |
| 46              |
| 47              |
| 48              |
| 49              |
| 50              |
| 51              |
| 52              |

---
**Query #3** <br>
How many total transactions were there for each year in the dataset?
```sql
    SELECT calendar_year,
    	   SUM(transactions) as total_transactions
    FROM clean_weekly_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;
```
| calendar_year | total_transactions |
| ------------- | ------------------ |
| 2018          | 346406460          |
| 2019          | 365639285          |
| 2020          | 375813651          |

---
**Query #4** <br>
What is the total sales for each region for each month?
```sql
    SELECT month_number,
    	   region,
           SUM(sales) as Total_Sales
    FROM clean_weekly_sales
    GROUP BY month_number, region
    ORDER BY month_number, region;
```
| month_number | region        | total_sales |
| ------------ | ------------- | ----------- |
| 3            | AFRICA        | 567767480   |
| 3            | ASIA          | 529770793   |
| 3            | CANADA        | 144634329   |
| 3            | EUROPE        | 35337093    |
| 3            | OCEANIA       | 783282888   |
| 3            | SOUTH AMERICA | 71023109    |
| 3            | USA           | 225353043   |
| 4            | AFRICA        | 1911783504  |
| 4            | ASIA          | 1804628707  |
| 4            | CANADA        | 484552594   |
| 4            | EUROPE        | 127334255   |
| 4            | OCEANIA       | 2599767620  |
| 4            | SOUTH AMERICA | 238451531   |
| 4            | USA           | 759786323   |
| 5            | AFRICA        | 1647244738  |
| 5            | ASIA          | 1526285399  |
| 5            | CANADA        | 412378365   |
| 5            | EUROPE        | 109338389   |
| 5            | OCEANIA       | 2215657304  |
| 5            | SOUTH AMERICA | 201391809   |
| 5            | USA           | 655967121   |
| 6            | AFRICA        | 1767559760  |
| 6            | ASIA          | 1619482889  |
| 6            | CANADA        | 443846698   |
| 6            | EUROPE        | 122813826   |
| 6            | OCEANIA       | 2371884744  |
| 6            | SOUTH AMERICA | 218247455   |
| 6            | USA           | 703878990   |
| 7            | AFRICA        | 1960219710  |
| 7            | ASIA          | 1768844756  |
| 7            | CANADA        | 477134947   |
| 7            | EUROPE        | 136757466   |
| 7            | OCEANIA       | 2563459400  |
| 7            | SOUTH AMERICA | 235582776   |
| 7            | USA           | 760331754   |
| 8            | AFRICA        | 1809596890  |
| 8            | ASIA          | 1663320609  |
| 8            | CANADA        | 447073019   |
| 8            | EUROPE        | 122102995   |
| 8            | OCEANIA       | 2432313652  |
| 8            | SOUTH AMERICA | 221166052   |
| 8            | USA           | 712002790   |
| 9            | AFRICA        | 276320987   |
| 9            | ASIA          | 252836807   |
| 9            | CANADA        | 69067959    |
| 9            | EUROPE        | 18877433    |
| 9            | OCEANIA       | 372465518   |
| 9            | SOUTH AMERICA | 34175583    |
| 9            | USA           | 110532368   |

---
**Query #5** <br>
What is the total count of transactions for each platform ?
```sql
    SELECT platform,
    	   SUM(transactions) as total_transactions
    FROM clean_weekly_sales
    GROUP BY platform;
```
| platform | total_transactions |
| -------- | ------------------ |
| Shopify  | 5925169            |
| Retail   | 1081934227         |

---
**Query #6** <br>
What is the percentage of sales for Retail vs Shopify for each month ?
```sql
    WITH platform_sales AS (
    SELECT calendar_year,
    	   month_number,
           platform,
           SUM(sales) as total_sales
    FROM clean_weekly_sales
    GROUP BY calendar_year, month_number, platform, sales
    ORDER BY calendar_year, month_number, platform
    )
    SELECT calendar_year,
    	   month_number,
           ROUND(100 * SUM(
                 CASE WHEN platform = 'Retail' THEN total_sales
                 ELSE NULL
                 END) / SUM(total_sales) , 2) as  retail_percentage,
           ROUND(100 * SUM(
                 CASE WHEN platform = 'Shopify' THEN total_sales
                 ELSE NULL
                 END) / SUM(total_sales) , 2) as  shopify_percentage
    FROM platform_sales
    GROUP BY calendar_year, month_number
    ORDER BY calendar_year, month_number;
```
| calendar_year | month_number | retail_percentage | shopify_percentage |
| ------------- | ------------ | ----------------- | ------------------ |
| 2018          | 3            | 97.92             | 2.08               |
| 2018          | 4            | 97.93             | 2.07               |
| 2018          | 5            | 97.73             | 2.27               |
| 2018          | 6            | 97.76             | 2.24               |
| 2018          | 7            | 97.75             | 2.25               |
| 2018          | 8            | 97.71             | 2.29               |
| 2018          | 9            | 97.68             | 2.32               |
| 2019          | 3            | 97.71             | 2.29               |
| 2019          | 4            | 97.80             | 2.20               |
| 2019          | 5            | 97.52             | 2.48               |
| 2019          | 6            | 97.42             | 2.58               |
| 2019          | 7            | 97.35             | 2.65               |
| 2019          | 8            | 97.21             | 2.79               |
| 2019          | 9            | 97.09             | 2.91               |
| 2020          | 3            | 97.30             | 2.70               |
| 2020          | 4            | 96.96             | 3.04               |
| 2020          | 5            | 96.71             | 3.29               |
| 2020          | 6            | 96.80             | 3.20               |
| 2020          | 7            | 96.67             | 3.33               |
| 2020          | 8            | 96.51             | 3.49               |

---
**Query #7** <br>
What is the percentage of sales by demographic for each year in the dataset ?
```sql
    WITH demographic_sales AS (
    SELECT calendar_year,
    	   demographic,
           SUM(sales) AS total_sales
    FROM clean_weekly_sales
    GROUP BY calendar_year, demographic
    ORDER BY calendar_year, demographic
    )
    SELECT calendar_year,
    	   ROUND(100 * SUM(
                 CASE WHEN demographic = 'Couples' THEN total_sales
                 ELSE NULL
                 END) / SUM(total_sales) , 2) as  couples_percentage,
           ROUND(100 * SUM(
                 CASE WHEN demographic = 'Families' THEN total_sales
                 ELSE NULL
                 END) / SUM(total_sales) , 2) as  family_percentage,
           ROUND(100 * SUM(
                 CASE WHEN demographic = 'unknown' THEN total_sales
                 ELSE NULL
                 END) / SUM(total_sales) , 2) as  unknown_percentage
    FROM demographic_sales
    GROUP BY calendar_year
    ORDER BY calendar_year;
```
| calendar_year | couples_percentage | family_percentage | unknown_percentage |
| ------------- | ------------------ | ----------------- | ------------------ |
| 2018          | 26.38              | 31.99             | 41.63              |
| 2019          | 27.28              | 32.47             | 40.25              |
| 2020          | 28.72              | 32.73             | 38.55              |

---
**Query #8** <br>
Which age_band and demographic values contribute the most to Retail sales ?
```sql
    SELECT age_band,
    	   demographic,
           SUM(sales) as total_sales,
           ROUND(100 * SUM(sales) / SUM(SUM(sales)) OVER (), 2) AS sales_percentage
    FROM clean_weekly_sales
    WHERE platform = 'Retail'
    GROUP BY age_band, demographic
    ORDER BY total_sales DESC;
```
| age_band     | demographic | total_sales | sales_percentage |
| ------------ | ----------- | ----------- | ---------------- |
| unknown      | unknown     | 16067285533 | 40.52            |
| Retirees     | Families    | 6634686916  | 16.73            |
| Retirees     | Couples     | 6370580014  | 16.07            |
| Middle Aged  | Families    | 4354091554  | 10.98            |
| Young Adults | Couples     | 2602922797  | 6.56             |
| Middle Aged  | Couples     | 1854160330  | 4.68             |
| Young Adults | Families    | 1770889293  | 4.47             |

---
**Query #9** <br>
Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
> We cannot use the avg_transaction column to find the average transaction size because this column calculates the average for each sale and transaction for each row but we would need the whole dataset to calculate for each year on each platform , the following query is how to correctly find the average transaction size for each year for each platform :
```sql
    SELECT calendar_year,
    	   platform,
           SUM(sales)/SUM(transactions) AS avg_transactions_per_platform
    FROM clean_weekly_sales
    GROUP BY calendar_year, platform
    ORDER BY calendar_year, avg_transactions_per_platform DESC;
```
| calendar_year | platform | avg_transactions_per_platform |
| ------------- | -------- | ----------------------------- |
| 2018          | Shopify  | 192                           |
| 2018          | Retail   | 36                            |
| 2019          | Shopify  | 183                           |
| 2019          | Retail   | 36                            |
| 2020          | Shopify  | 179                           |
| 2020          | Retail   | 36                            |
