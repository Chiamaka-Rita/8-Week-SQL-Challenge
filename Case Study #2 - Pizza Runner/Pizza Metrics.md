## Pizza Metrics
### Q1. How many pizzas were ordered?
``` SQL
SELECT count(pizza_id)
FROM new_customer_orders;
```
| count |
|-------|
| 14    |

### Q2. How many unique customer orders were made?
```SQL
SELECT COUNT(DISTINCT order_id) unique_customer_orders
FROM new_customer_orders;
```
| unique_customer_orders |
|------------------------|
| 10                     |

### Q3. How many successful orders were delivered by each runner?
``` SQL
SELECT runner_id, COUNT(order_id) successful_orders
FROM new_runner_orders
WHERE cancellation = ''
GROUP BY runner_id;
```
| runner_id | successful_orders |
|-----------|-------------------|
| 1         | 4                 |
| 2         | 3                 |
| 3         | 1                 |

### Q4. How many of each type of pizza was delivered?
``` SQL
SELECT p.pizza_id, pizza_name, COUNT(cancellation)
FROM new_runner_orders r
JOIN new_customer_orders c
ON r.order_id = c.order_id
JOIN pizza_runner.pizza_names p
ON p.pizza_id = c.pizza_id
WHERE cancellation = ''
GROUP BY p.pizza_id, pizza_name;
```
| pizza_id | pizza_name | count |
|----------|------------|-------|
| 1        | Meatlovers | 9     |
| 2        | Vegetarian | 3     |

### Q5. How many Vegetarian and Meatlovers were ordered by each customer?
``` SQL
SELECT customer_id, pizza_name, COUNT(p.pizza_id) pizza_ordered
FROM new_runner_orders r
JOIN new_customer_orders c
ON r.order_id = c.order_id
JOIN pizza_runner.pizza_names p
ON p.pizza_id = c.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id;
```
| customer_id | pizza_name | pizza_ordered |
|-------------|------------|---------------|
| 101         | Meatlovers | 2             |
| 101         | Vegetarian | 1             |
| 102         | Meatlovers | 2             |
| 102         | Vegetarian | 1             |
| 103         | Meatlovers | 3             |
| 103         | Vegetarian | 1             |
| 104         | Meatlovers | 3             |
| 105         | Vegetarian | 1             |

### Q6. What was the maximum number of pizzas delivered in a single order
``` SQL
SELECT order_id, MAX(pizza_count) maximum_pizza_ordered
FROM (SELECT r.order_id, count(pizza_id) pizza_count
FROM new_runner_orders r
JOIN new_customer_orders c
ON r.order_id = c.order_id
WHERE cancellation = ''
GROUP BY r.order_id) as table1
GROUP BY order_id
ORDER BY maximum_pizza_ordered DESC
LIMIT 1;
```
| order_id | maximum_pizza_ordered |
|----------|-----------------------|
| 4        | 3                     |

### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes
``` SQL
SELECT customer_id, SUM(pizzas_with_change) pizzas_with_change, SUM(pizzas_without_change) pizzas_without_change
FROM(SELECT order_id, customer_id, CASE WHEN LENGTH(exclusions) >= 1 OR LENGTH(extras) >= 1 THEN count(pizza_id) ELSE '0' END AS pizzas_with_change, CASE WHEN LENGTH(exclusions) < 1 AND LENGTH(extras) < 1 THEN count(pizza_id) ELSE '0' END AS pizzas_without_change
FROM new_customer_orders
GROUP BY customer_id, exclusions, extras, order_id) AS table1
JOIN new_runner_orders r
ON r.order_id = table1.order_id
WHERE cancellation = ''
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | pizza_with_change | pizzas_without_change |
|-------------|-------------------|-----------------------|
| 101         | 0                 | 2                     |
| 102         | 0                 | 3                     |
| 103         | 3                 | 0                     |
| 104         | 2                 | 1                     |
| 105         | 1                 | 0                     |

### Q8. How many pizzas were delivered that had both exclusions and extras?
``` SQL
SELECT customer_id, count(pizza_id)
FROM new_customer_orders n
JOIN new_runner_orders r
ON r.order_id = n.order_id
WHERE LENGTH(exclusions) >= 1 AND LENGTH(extras) >= 1 AND cancellation = ''
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | count |
|-------------|-------|
| 104         | 1     |
|             |       |

### Q9. What was the total volume of pizzas ordered for each hour of the day?
``` SQL
SELECT EXTRACT (HOUR FROM order_time) hour_of_day, count(pizza_id) pizzas_ordered 
FROM new_customer_orders
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
| hour_of_day | pizzas_ordered |
|-------------|----------------|
| 11          | 1              |
| 13          | 3              |
| 18          | 3              |
| 19          | 1              |
| 21          | 3              |
| 23          | 3              |

### Q10. What was the volume of orders for each day of the week?
``` SQL
SELECT TO_CHAR(order_time, 'Day') day_of_the_week, count(pizza_id) pizzas_ordered 
FROM new_customer_orders
GROUP BY days
ORDER BY days;
```
| day_of_the_week | pizzas_ordered |
|-----------------|----------------|
| Friday          | 1              |
| Saturday        | 5              |
| Thursday        | 3              |
| Wednesday       | 5              |
