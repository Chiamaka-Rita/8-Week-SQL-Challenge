### Q1. HOw many customers has Foodie-Fi ever had?
``` SQL
SELECT COUNT(distinct(customer_id))
FROM foodie_fi.subscriptions;
```
| num_of_customers |
| ---------------- |
| 1000             |

### Q2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?
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

### Q3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
```SQL
WITH table1 AS (SELECT EXTRACT('month' FROM start_date) AS month_of_the_year, COUNT(start_date) AS start_date, plan_name
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE EXTRACT('year' FROM start_date) = '2021'
GROUP BY plan_name, start_date)

SELECT plan_name, SUM(start_date) num_of_events
FROM table1
GROUP BY plan_name
ORDER BY num_of_events;
```
| plan_name     | num_of_events |
| ------------- | ------------- |
| basic monthly | 8             |
| pro monthly   | 60            |
| pro annual    | 63            |
| churn         | 71            |

### Q4. What is the customer count and percentage of customer plans who have churned rounded to 1 decimal place?
``` SQL
SELECT COUNT(DISTINCT customer_id) AS churned_customers, ROUND(100.0 * COUNT(DISTINCT customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 1) AS churned_percentage
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE plan_name = 'churn';
```
| churned_customers | churned_percentage |
| ----------------- | ------------------ |
| 307               | 30.7               |

### Q5. How many customers have churned straight after their inintial free trial - what percentage is this rounded to the nearest whole number?
```SQL
WITH table1 AS (SELECT customer_id, plan_name trial_plan_name, start_date trial_start_date, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date)
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE plan_name = 'trial'),

table2 AS (SELECT customer_id, plan_name AS churn_plan_name, start_date churn_start_date, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date)
FROM foodie_fi.subscriptions fs
JOIN foodie_fi.plans fp
ON fs.plan_id = fp.plan_id
WHERE plan_name = 'churn'),

table3 AS (SELECT t1.customer_id, trial_plan_name, trial_start_date, churn_plan_name, churn_start_date, CASE WHEN (trial_start_date + 7) = churn_start_date THEN 'churned' ELSE 'subscribed' END AS churned_after_trial
FROM table1 t1
JOIN table2 t2
ON t1.customer_id = t2.customer_id)

SELECT count(churned_after_trial) churned_after_trial, round(100 * count(churned_after_trial)/ (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)) churned_percentage
FROM table3
WHERE churned_after_trial = 'churned'
```
| churned_after_trial | churned_percentage |
| ------------------- | ------------------ |
| 92                  | 9                  |

### Q6. What is the number an dpercentage of the customer plans after their initial free trial?
```SQL
WITH table2 as (WITH table1 AS (SELECT customer_id, plan_name, ROW_NUMBER() OVER(PARTITION BY fs.customer_id ORDER BY fs.plan_id)
FROM foodie_fi.plans fp
JOIN foodie_fi.subscriptions fs
ON fp.plan_id = fs.plan_id
GROUP BY plan_name, customer_id, fs.plan_id, start_date)

SELECT plan_name, COUNT(row_number) customers_plan_count
FROM table1
WHERE row_number = 2 AND plan_name != 'trial'
GROUP BY plan_name)

SELECT *, ROUND(sum(customers_plan_count) * 100.0/SUM(SUM(customers_plan_count)) OVER (), 1) AS percentage
FROM table2
GROUP BY plan_name, customers_plan_count;
```
| plan_name     | customers_plan_count | percentage |
| ------------- | -------------------- | ---------- |
| basic monthly | 546                  | 54.6       |
| churn         | 92                   | 9.2        |
| pro annual    | 37                   | 3.7        |
| pro monthly   | 325                  | 32.5       |

