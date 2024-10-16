### Q1. What are the standard ingredients for each pizza?
```SQL
WITH table2 AS (WITH table1 AS (SELECT pizza_id, UNNEST(STRING_TO_ARRAY(toppings, ',')::integer[]) toppings
               FROM pizza_runner.pizza_recipes)
               
SELECT r.pizza_id, n.pizza_name, ARRAY_TO_STRING(ARRAY[t1.toppings], ',') topping_id,  topping_name
FROM pizza_runner.pizza_recipes r
JOIN pizza_runner.pizza_names n
ON r.pizza_id = n.pizza_id
JOIN table1 t1
ON t1.pizza_id = r.pizza_id
JOIN pizza_runner.pizza_toppings pt
ON t1.toppings = pt.topping_id)

SELECT pizza_name, ARRAY_TO_STRING(ARRAY_AGG(topping_name), ', ')
FROM table2
GROUP BY pizza_name;
```
<pr>-- unnest expands an array to a set of rows <br>
<pr>-- splits string into array elements using supplied delimiter and optional null string <br>
<pr>-- array_agg aggregates the values of the topping_name into an array for each pizza name <br>
<pr>-- array_to_string converts the array in to string with a space as a delimiter between the values <br>

### Q2. What was the most commonly added extra?
```SQL
WITH table1 as (SELECT order_id, UNNEST(string_to_array(extras, ',')::integer[]) extras
FROM new_customer_orders
WHERE extras != ''),

table2 AS (SELECT MODE() WITHIN GROUP (ORDER BY extras) most_common_extra
FROM table1
          group by order_id)

SELECT topping_name, count(topping_name) frequency
FROM table2
JOIN pizza_runner.pizza_toppings pt
ON pt.topping_id = table2.most_common_extra
group by topping_name;
```
### Q3. What was the most common exclusion?
```SQL
SELECT topping_name, most_common_exclusion
FROM(SELECT MODE() WITHIN GROUP (ORDER BY exclusions) most_common_exclusion
FROM (SELECT order_id, UNNEST(string_to_array(exclusions, ',')::integer[]) exclusions
FROM new_customer_orders
WHERE exclusions != ''
ORDER BY order_id) AS table1) as table2
JOIN pizza_runner.pizza_toppings pt
ON pt.topping_id = table2.most_common_exclusion;
```

