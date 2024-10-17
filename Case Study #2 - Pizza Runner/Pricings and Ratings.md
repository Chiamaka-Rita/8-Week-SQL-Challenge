### Q1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```SQL
SELECT COUNT(pizza_id)amount_sold, SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) AS revenue_usd
FROM new_customer_orders nc
JOIN new_runner_orders nr
ON nc.order_id = nr.order_id
WHERE cancellation = ''
```
| amount_sold | revenue_usd |
| ----------- | ----------- |
| 12          | 138         |

### Q2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
```SQL
SELECT COUNT(pizza_id) amount_sold, SUM(CASE WHEN pizza_id = 1 AND LENGTH(regexp_replace(extras, '[^0-9]','', 'g')) > 0 THEN 12 + (1* LENGTH(regexp_replace(extras, '[^0-9]','', 'g'))) WHEN pizza_id = 1 AND LENGTH(extras) <= 0 THEN 12 WHEN pizza_id = 2 AND LENGTH(regexp_replace(extras, '[^0-9]','', 'g')) > 0 THEN 10 + (1* LENGTH(regexp_replace(extras, '[^0-9]','', 'g'))) WHEN pizza_id = 2 AND LENGTH(regexp_replace(extras, '[^0-9]','', 'g')) <= 0 THEN 10 END ) AS revenue_usd
FROM new_customer_orders nc
JOIN new_runner_orders nr
ON nc.order_id = nr.order_id
WHERE cancellation = ''
```
| amount_sold | revenue_usd |
| ----------- | ----------- |
| 12          | 142         |

### Q3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```SQL
SET search_path = pizza_runner;
DROP TABLE IF EXISTS runner_rating;

CREATE TABLE runner_rating (order_id int primary key, customer_id int, runner_id int, rating int);

INSERT INTO runner_rating (order_id, customer_id, runner_id, rating)
VALUES (1, 101, 1, 5),
	   (2, 101, 1, 3),
       (3, 102, 1, 4),
       (4, 103, 2, 4),
       (5, 104, 3, 5),
       (7, 105, 2, 4),
       (8, 102, 2, 3),
       (10, 104, 1, 4);
SELECT *
FROM runner_rating;
```
| order_id | customer_id | runner_id | rating |
| -------- | ----------- | --------- | ------ |
| 1        | 101         | 1         | 5      |
| 2        | 101         | 1         | 3      |
| 3        | 102         | 1         | 4      |
| 4        | 103         | 2         | 4      |
| 5        | 104         | 3         | 5      |
| 7        | 105         | 2         | 4      |
| 8        | 102         | 2         | 3      |
| 10       | 104         | 1         | 4      |

### Q4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas

```SQL
SET search_path = pizza_runner;
DROP TABLE IF EXISTS runner_rating;

CREATE TABLE runner_rating (order_id int primary key, customer_id int, runner_id int, rating int);

INSERT INTO runner_rating (order_id, customer_id, runner_id, rating)
VALUES (1, 101, 1, 5),
	   (2, 101, 1, 3),
       (3, 102, 1, 4),
       (4, 103, 2, 4),
       (5, 104, 3, 5),
       (7, 105, 2, 4),
       (8, 102, 2, 3),
       (10, 104, 1, 4);

WITH table1 AS (SELECT nc.customer_id, nr.order_id, runner_id, order_time, pickup_time, duration_mins, ROUND((distance_km/duration_mins) * 60) avg_speed_km_hr, EXTRACT('minute' FROM (date_trunc('minute', (pickup_time - order_time)))) time_difference
FROM new_customer_orders nc
JOIN new_runner_orders nr
ON nc.order_id = nr.order_id
WHERE cancellation = ''),

table2 AS (SELECT order_id, count(pizza_id) num_pizzas
FROM new_customer_orders
GROUP BY order_id)

SELECT t1.customer_id, t1.order_id, t1.runner_id, rating, order_time, pickup_time, time_difference, avg_speed_km_hr, duration_mins, num_pizzas
FROM table1 t1
JOIN table2 t2
ON t1.order_id = t2.order_id
JOIN runner_rating rr
ON rr.order_id = t1.order_id
ORDER BY order_id;
```
| customer_id | order_id | runner_id | rating | order_time               | pickup_time              | time_difference | avg_speed_km_hr | duration_mins | num_pizzas |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------------ | --------------- | --------------- | ------------- | ---------- |
| 101         | 1        | 1         | 5      | 2020-01-01T18:05:02.000Z | 2020-01-01T18:15:34.000Z | 10              | 38              | 32            | 1          |
| 101         | 2        | 1         | 3      | 2020-01-01T19:00:52.000Z | 2020-01-01T19:10:54.000Z | 10              | 44              | 27            | 1          |
| 102         | 3        | 1         | 4      | 2020-01-02T23:51:23.000Z | 2020-01-03T00:12:37.000Z | 21              | 40              | 20            | 2          |
| 103         | 4        | 2         | 4      | 2020-01-04T13:23:46.000Z | 2020-01-04T13:53:03.000Z | 29              | 35              | 40            | 3          |
| 104         | 5        | 3         | 5      | 2020-01-08T21:00:29.000Z | 2020-01-08T21:10:57.000Z | 10              | 40              | 15            | 1          |
| 105         | 7        | 2         | 4      | 2020-01-08T21:20:29.000Z | 2020-01-08T21:30:45.000Z | 10              | 60              | 25            | 1          |
| 102         | 8        | 2         | 3      | 2020-01-09T23:54:33.000Z | 2020-01-10T00:15:02.000Z | 20              | 94              | 15            | 1          |
| 104         | 10       | 1         | 4      | 2020-01-11T18:34:49.000Z | 2020-01-11T18:50:20.000Z | 15              | 60              | 10            | 2          |

### Q5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```SQL
WITH table1 AS (SELECT nc.order_id, pizza_id, runner_id, distance_km, CASE WHEN pizza_id = 1 then 12 
WHEN pizza_id = 2 then 10 end as cost_usd, (distance_km *0.30) runner_payment_usd
FROM new_runner_orders nr
JOIN new_customer_orders nc
ON nr.order_id = nc.order_id
WHERE cancellation = '')
​
SELECT ROUND(SUM(cost_usd) - SUM(runner_payment_usd)::integer, 2) revenue
FROM table1;
```
| revenue |
| ------- |
| 73      |
