# Runner and Customer Experience

### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```SQL
SELECT date_trunc('week', registration_date) + INTERVAL '4 days' AS registration_week, COUNT(runner_id) num_runners
FROM pizza_runner.runners
GROUP BY registration_week
ORDER BY registration_week;
```
| registration_week        | num_runners |
|--------------------------|-------------|
| 2021-01-01T00:00:00.000Z | 2           |
| 2021-01-08T00:00:00.000Z | 1           |
| 2021-01-15T00:00:00.000Z | 1           |


### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```SQL
SELECT runner_id, EXTRACT('minute' FROM AVG(date_trunc('minute', (pickup_time - order_time)))) avg_arrival_time_mins
FROM new_runner_orders r
JOIN new_customer_orders c
ON c.order_id = r.order_id
WHERE pickup_time IS NOT NULL 
GROUP BY runner_id
ORDER BY runner_id;
```
| runner_id | avg_arrival_time_mins |
|-----------|-----------------------|
| 1         | 15                    |
| 2         | 23                    |
| 3         | 10                    |

### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```SQL
WITH table1 as (SELECT c.order_id, count(pizza_id) num_pizzas_in_order, EXTRACT('minute' FROM (pickup_time - order_time)) prep_time
FROM new_customer_orders c
JOIN new_runner_orders r
ON c.order_id = r.order_id
WHERE pickup_time IS NOT NULL
GROUP BY c.order_id, prep_time)

SELECT num_pizzas_in_order, AVG(prep_time) prep_time
FROM table1
GROUP BY num_pizzas_in_order
ORDER BY num_pizzas_in_order;
```
| num_pizzas_in_order | prep_time |
|---------------------|---------------|
| 1                   | 12            |
| 2                   | 18            |
| 3                   | 29            |

### Q4. What was the average distance travelled for each customer?
```SQL
SELECT customer_id, round(avg(distance_km)) avg_distance_travelled
FROM new_runner_orders r
JOIN new_customer_orders c
ON c.order_id = r.order_id
WHERE pickup_time is not null
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | avg_distance_travelled |
|-------------|------------------------|
| 101         | 20                     |
| 102         | 17                     |
| 103         | 23                     |
| 104         | 10                     |
| 105         | 25                     |

### Q5. What was the difference between the longest and shortest delivery times for all orders?
```SQL
SELECT MAX(duration_mins) - MIN(duration_mins) max_time_diff
FROM new_runner_orders;
```
| max_time_diff |
|---------------|
| 30            |

### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```SQL
WITH table1 AS (SELECT runner_id, order_id, round((distance_km/duration_mins) * 60) speed_km_hr
FROM new_runner_orders
WHERE duration_mins IS NOT NULL)

SELECT *
FROM table1
ORDER BY runner_id, order_id;
```
| runner_id | order_id | speed_km_hr |
|-----------|----------|-------------|
| 1         | 1        | 38          |
| 1         | 2        | 44          |
| 1         | 3        | 40          |
| 1         | 10       | 60          |
| 2         | 4        | 35          |
| 2         | 7        | 60          |
| 2         | 8        | 94          |
| 3         | 5        | 40          |


### Q7. What is the successful delivery percentage for each runner?
```SQL
WITH table1 AS (SELECT runner_id, COUNT(cancellation) total_order
FROM new_runner_orders                       
GROUP BY runner_id),
table2 AS (SELECT runner_id, COUNT(cancellation) completed_order
           FROM new_runner_orders
           WHERE duration_mins IS NOT null
           GROUP BY runner_id)
SELECT t2.runner_id, total_order, completed_order, round((completed_order::numeric/total_order)*100) success_percentage
FROM table2 t2
FULL JOIN table1 t1
ON t1.runner_id = t2.runner_id
ORDER BY runner_id;
```
| runner_id | total_order | completed_order | success_percentage |
|-----------|-------------|-----------------|--------------------|
| 1         | 4           | 4               | 100                |
| 2         | 4           | 3               | 75                 |
| 3         | 2           | 1               | 50                 |
