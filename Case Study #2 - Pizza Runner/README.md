# Problem Statement
Danny who is interested in expanding his new pizza empire decided to include runners to help with delivering fresh pizza to his cutomers. Danny being an experienced data scientist had collected data and developed an entity relationship diagram of his database. However, he would like assistance with cleaning and transforming his data to better direct his runners and optimize his business operations.
# Dataset
The dataset provided for this analysis includes;
1. runners
2. customer_orders
3. runner_orders
4. pizza_names
6. pizza_recipes
7. pizza_toppings

Click [here](https://8weeksqlchallenge.com/case-study-2/) for more infromation about this challenge
# Case Study Questions
The case study questions are broken up by area of focus which includes:
- Pizza Metrics
- Runner and Customer Experience
- Ingredient Optimisation
- Pricing and Ratings
- Bonus DML Challenges (DML = Data Manipulation Language)
# Solutions
## Data Cleaning
Some inconsistencies were observed within the dataset provided in the runner_orders and customer_orders tables. Both tables contained nulls and NaN, runner_orders table also had the wrong data types for some of its column. Click [here](https://8weeksqlchallenge.com/case-study-2/) to view the table. So therefore, for the purpose of better analysis, these tables have to be cleaned. For this data preprocessing, temporary tables were created so as to avoid making changes to the original dataset provided.

### Cleaning of the runner_orders table
```SQL
-- A temp table was created for this section so as not to alter the original dataset 
CREATE TEMP TABLE new_runner_orders AS
SELECT * FROM pizza_runner.runner_orders;

-- Remove 'minutes' within the duration column for consistency
UPDATE new_runner_orders
SET duration = LEFT(duration,2)
WHERE duration LIKE '%min%';

-- Remove the 'km' within the duration column for consistency
UPDATE new_runner_orders
SET distance = SUBSTRING(distance, 1, position('k' in distance) - 1)
WHERE distance LIKE '%km';

-- Replace nulls in cancellation column to blanks
UPDATE new_runner_orders
SET cancellation = ''
WHERE cancellation IS NULL OR cancellation LIKE '%null';

--  Replace nulls in pickup_time column to blanks
UPDATE new_runner_orders
SET pickup_time = ''
WHERE pickup_time IS NULL OR pickup_time LIKE '%null'; 

--  Replace nulls in distance column to blanks
UPDATE new_runner_orders
SET distance = ''
WHERE distance IS NULL OR distance LIKE '%null';

--  Replace nulls in duration columns to blanks
UPDATE new_runner_orders
SET duration = ''
WHERE duration IS NULL OR duration LIKE '%null';

/* Incorrect Data type, change pickup_time data type to timestamp*/
-- Create a temporary timestamp column
ALTER TABLE new_runner_orders 
ADD COLUMN create_time_holder TIMESTAMP NULL; 

-- Copy casted value over to the temporary time holder column
UPDATE new_runner_orders
SET create_time_holder = pickup_time::TIMESTAMP
WHERE pickup_time != '';

-- Modify original column using the temporary column
ALTER TABLE new_runner_orders 
ALTER COLUMN pickup_time TYPE TIMESTAMP USING create_time_holder;

-- Drop temporary column
ALTER TABLE new_runner_orders 
DROP COLUMN create_time_holder;

-- Clean column duration of incorrect data type 
ALTER TABLE new_runner_orders  
ALTER COLUMN duration TYPE INTEGER 
USING NULLIF(duration, '')::INTEGER;

-- Clean column distance of incorrect data type 
ALTER TABLE new_runner_orders 
ALTER COLUMN distance TYPE FLOAT
USING NULLIF(distance, '')::FLOAT;

-- Edit column name for readability
ALTER TABLE new_runner_orders 
RENAME COLUMN distance TO distance_km;

ALTER TABLE new_runner_orders 
RENAME COLUMN duration TO duration_mins;
```
### Cleaning of the customers_orders table
``` SQL
/* A temp table was created for this section so as not to alter the original dataset */
CREATE TEMP TABLE new_customer_orders AS
SELECT * from pizza_runner.customer_orders;

UPDATE new_customer_orders
SET exclusions = ''
WHERE exclusions is null or exclusions like '%null';

UPDATE new_customer_orders
SET extras = ''
WHERE extras is null or extras like '%null';
```

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
