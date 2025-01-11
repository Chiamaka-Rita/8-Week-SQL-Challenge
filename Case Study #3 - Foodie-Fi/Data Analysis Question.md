### Q1. HOw many customers has Foodie-Fi ever had?
``` SQL
SELECT COUNT(distinct(customer_id))
FROM foodie_fi.subscriptions;
```
| num_of_customers |
| ---------------- |
| 1000             |

### Q2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```SQL
WITH table1 AS (SELECT TO_CHAR(TO_DATE(DATE_PART('month', start_date)::text, 'mm'), 'month') AS month_of_the_year, COUNT(start_date) monthly_distribution
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE plan_name = 'trial'
GROUP BY month_of_the_year)

SELECT *
FROM table1
ORDER BY TO_DATE(month_of_the_year, 'month');
```
| month_of_the_year | monthly_distribution |
| ----------------- | -------------------- |
| january           | 88                   |
| february          | 68                   |
| march             | 94                   |
| april             | 81                   |
| may               | 88                   |
| june              | 79                   |
| july              | 89                   |
| august            | 88                   |
| september         | 87                   |
| october           | 79                   |
| november          | 75                   |
| december          | 84                   |
