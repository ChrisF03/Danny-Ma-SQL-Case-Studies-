## A. Data Exploration and Cleansing

**1) Update the `fresh_segments.interest_metrics` table by modifying the month_year column to be a date data type with the start of the month**
```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER COLUMN month_year TYPE DATE 
USING TO_DATE(month_year, 'MM-YYYY');
```

---

**2) What is count of records in the `fresh_segments.interest_metrics` for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?**
```sql
SELECT month_year,
	   COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```
| month_year               | count |
| ------------------------ | ----- |
| NULL                     | 1194  |
| 2018-07-01T00:00:00.000Z | 729   |
| 2018-08-01T00:00:00.000Z | 767   |
| 2018-09-01T00:00:00.000Z | 780   |
| 2018-10-01T00:00:00.000Z | 857   |
| 2018-11-01T00:00:00.000Z | 928   |
| 2018-12-01T00:00:00.000Z | 995   |
| 2019-01-01T00:00:00.000Z | 973   |
| 2019-02-01T00:00:00.000Z | 1121  |
| 2019-03-01T00:00:00.000Z | 1136  |
| 2019-04-01T00:00:00.000Z | 1099  |
| 2019-05-01T00:00:00.000Z | 857   |
| 2019-06-01T00:00:00.000Z | 824   |
| 2019-07-01T00:00:00.000Z | 864   |
| 2019-08-01T00:00:00.000Z | 1149  |

---

**3) What do you think we should do with these null values in the `fresh_segments.interest_metrics` ?**

- First, lets take a closer look at the null values in the table below:
```sql
SELECT *
FROM fresh_segments.interest_metrics
WHERE month_year IS NULL
LIMIT 5;
```
| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
| ------ | ----- | ---------- | ----------- | ----------- | ----------- | ------- | ------------------ |
| NULL   | NULL  | NULL       | NULL        | 6.12        | 2.85        | 43      | 96.4               |
| NULL   | NULL  | NULL       | NULL        | 7.13        | 2.84        | 45      | 96.23              |
| NULL   | NULL  | NULL       | NULL        | 6.82        | 2.84        | 45      | 96.23              |
| NULL   | NULL  | NULL       | NULL        | 5.96        | 2.83        | 47      | 96.06              |
| NULL   | NULL  | NULL       | NULL        | 7.73        | 2.82        | 48      | 95.98              |

- Because none of the null month_year values have interest id's either, we are unable to provide a date or map them so I will remove them: 
```sql
DELETE
FROM fresh_segments.interest_metrics
WHERE month_year IS NULL;
```
```sql
SELECT COUNT(*) as null_count
FROM fresh_segments.interest_metrics
WHERE month_year IS NULL;
```
| null_count |
| ---------- |
| 0          |

---

**4) How many interest_id values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?**
```sql
SELECT COUNT(interest_id)
FROM fresh_segments.interest_metrics
WHERE interest_id::INTEGER NOT IN (SELECT DISTINCT id
                                   FROM fresh_segments.interest_map);
```
| count |
| ----- |
| 0     |

- Every interest_id from the interest_metrics table is in the interest_map table

```sql
SELECT COUNT(id)
FROM fresh_segments.interest_map
WHERE id::INTEGER NOT IN (SELECT DISTINCT interest_id::INTEGER
                          FROM fresh_segments.interest_metrics);
```
| count |
| ----- |
| 7     |

- There are 7 id's from the interest_map table that are not present in the interest_metrics table.

---

**5) Summarise the id values in the `fresh_segments.interest_map` by its total record count in this table**
```sql
SELECT COUNT(*) as total_records
FROM fresh_segments.interest_map;
```
| total_records |
| ------------- |
| 1209          |

---

**6) What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from fresh_segments.interest_map except from the id column.**

- An INNER JOIN (or just JOIN as this is the default JOIN anyway) should be used to perform our analysis, this provides us with all the rows from both tables where the condition is met (interest_id = 21246)
```sql
SELECT * 
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map ON
interest_metrics.interest_id::INTEGER = interest_map.id
WHERE interest_metrics.interest_id::INTEGER = 21246;
```
| _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking | id    | interest_name                    | interest_summary                                      | created_at               | last_modified            |
| ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ | ----- | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 21246       | 2.26        | 0.65        | 722     | 0.96               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 8      | 2018  | 2018-08-01T00:00:00.000Z | 21246       | 2.13        | 0.59        | 765     | 0.26               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 9      | 2018  | 2018-09-01T00:00:00.000Z | 21246       | 2.06        | 0.61        | 774     | 0.77               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 10     | 2018  | 2018-10-01T00:00:00.000Z | 21246       | 1.74        | 0.58        | 855     | 0.23               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 11     | 2018  | 2018-11-01T00:00:00.000Z | 21246       | 2.25        | 0.78        | 908     | 2.16               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 12     | 2018  | 2018-12-01T00:00:00.000Z | 21246       | 1.97        | 0.7         | 983     | 1.21               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 1      | 2019  | 2019-01-01T00:00:00.000Z | 21246       | 2.05        | 0.76        | 954     | 1.95               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 2      | 2019  | 2019-02-01T00:00:00.000Z | 21246       | 1.84        | 0.68        | 1109    | 1.07               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 3      | 2019  | 2019-03-01T00:00:00.000Z | 21246       | 1.75        | 0.67        | 1123    | 1.14               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 4      | 2019  | 2019-04-01T00:00:00.000Z | 21246       | 1.58        | 0.63        | 1092    | 0.64               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |

---

**7) Are there any records in your joined table where the month_year value is before the created_at value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?**
```sql
SELECT COUNT (*) 
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map ON
interest_metrics.interest_id::INTEGER = interest_map.id
WHERE interest_metrics.month_year < interest_map.created_at;
```
| count |
| ----- |
| 188   |

- There are 188 records where the month_year is before the created_at date but since the created_at column also has a day value and the month_year column defaults to 1 for the day we will cross-reference by comparing only the month value from the created_at column with with month_year column. As long as the month_year month is equal to or greater than created_at, the record should be considered valid.

```sql
SELECT COUNT (*) 
FROM fresh_segments.interest_metrics
JOIN fresh_segments.interest_map ON
interest_metrics.interest_id::INTEGER = interest_map.id
WHERE interest_metrics.month_year < DATE_TRUNC('mon', interest_map.created_at::DATE);
```
| count |
| ----- |
| 0     |
