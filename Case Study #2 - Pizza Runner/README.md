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

# Data Cleaning
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
# Solutions
### [A. Pizza Metrics](https://github.com/Chiamaka-Rita/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Pizza%20Metrics.md)
