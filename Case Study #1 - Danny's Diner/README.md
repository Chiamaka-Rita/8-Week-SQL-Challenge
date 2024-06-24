## Problem Statement
Danny a restaurant owner want to use data to answer simple questions questions about his customers. The insights from these analysis will help him decide if he should expand his exisiting customer loyalty program. Also, he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

## Dataset
The three key datasets for this analysis are as shown below;
* Sales
* Menu
* Members

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
3. How many days has each customer visited the restaurant?
4. What was the first item from the menu purchased by each customer?
5. What is the most purchased item on the menu and how many times was it purchased by all customers?
6. Which item was the most popular for each customer?
7. Which item was purchased first by the customer after they became a member?
8. Which item was purchased just before the customer became a member?
9. What is the total items and amount spent for each member before they became a member?
10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Solutions
### Q1. What is the total amount each customer spent at the restaurant?
```SQL
SELECT customer_id, SUM(price) total_amount_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total_amount_spent |
|-------------|--------------------|
| A           | 76                 |
| B           | 74                 |
| C           | 36                 |


### Q2. How many days has each customer visited the restaurant?
```SQL
SELECT  customer_id, COUNT( DISTINCT order_date)
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | count |
|-------------|-------|
| A           | 4     |
| B           | 6     |
| C           | 2     |

### Q3. What was the first item from the menu purchased by each customer?
```SQL
SELECT customer_id, order_date, product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
on s.product_id = m.product_id
WHERE order_date = (SELECT date_trunc('month', MIN(order_date)) FROM dannys_diner.sales)
ORDER BY customer_id
```
| customer_id | order_date               | product_name |
|-------------|--------------------------|--------------|
| A           | 2021-01-01T00:00:00.000Z | sushi        |
| A           | 2021-01-01T00:00:00.000Z | curry        |
| B           | 2021-01-01T00:00:00.000Z | curry        |
| C           | 2021-01-01T00:00:00.000Z | ramen        |
| C           | 2021-01-01T00:00:00.000Z | ramen        |

### Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```SQL
SELECT s.product_id, product_name, COUNT(product_name) times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY times_purchased DESC
LIMIT 1
```
| product_id | product_name | times_purchased |
|------------|--------------|-----------------|
| 3          | ramen        | 8               |

### Q5. Which item was the most popular for each customer?
```SQL
SELECT customer_id, product_name, times_purchased
FROM(SELECT customer_id, product_name, times_purchased, rank() OVER (PARTITION BY customer_id ORDER BY times_purchased DESC)
FROM (SELECT s.customer_id, product_name, COUNT(product_name) times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, product_name) as sub_query1) as sub_query2
WHERE rank = 1
```
| customer_id | product_name | times_purchased |
|-------------|--------------|-----------------|
| A           | ramen        | 3               |
| B           | ramen        | 2               |
| B           | curry        | 2               |
| B           | sushi        | 2               |
| C           | ramen        | 3               |

### Q6. Which item was purchased first by the customer after they became a member?
```SQL
WITH table1 as (SELECT s.customer_id, join_date, order_date, product_id
FROM dannys_diner.members m
JOIN dannys_diner.sales s
ON s.customer_id = m.customer_id
WHERE join_date <= order_date),

table2 as (SELECT *, rank() over(partition by join_date order by order_date) 
FROM table1)

SELECT customer_id, join_date, order_date, product_name
FROM table2 t
JOIN dannys_diner.menu m
ON m.product_id = t.product_id
WHERE rank = 1
ORDER BY customer_id
```
| customer_id | join_date                | order_date               | product_name |
| ----------- | ------------------------ | ------------------------ | ------------ |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z | curry        |
| B           | 2021-01-09T00:00:00.000Z | 2021-01-11T00:00:00.000Z | sushi        |


### Q7. Which item was purchased just before the customer became a member?
```SQL
WITH table1 as (SELECT s.customer_id, join_date, order_date, product_id
FROM dannys_diner.members m
JOIN dannys_diner.sales s
ON s.customer_id = m.customer_id
WHERE join_date > order_date),

table2 as (SELECT *, RANK() over(PARTITION BY customer_id ORDER BY order_date desc) 
FROM table1)

SELECT customer_id, product_name
FROM table2 t
JOIN dannys_diner.menu m
ON t.product_id = m.product_id
WHERE rank = 1
ORDER BY customer_id
```

| customer_id | product_name |
|-------------|--------------|
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

### Q8. What is the total items and amount spent for each member before they became a member?
```SQL
WITH table1 as (SELECT s.customer_id, join_date, order_date, product_id
FROM dannys_diner.members m
JOIN dannys_diner.sales s
ON s.customer_id = m.customer_id
WHERE join_date > order_date)

SELECT customer_id, count(m.product_id) total_items, sum(price) total_amount_Spent 
FROM table1 t
JOIN dannys_diner.menu m
ON t.product_id = m.product_id
GROUP BY customer_id
ORDER BY customer_id
```
| customer_id | total_items | total_amount_spent |
|-------------|-------------|--------------------|
| A           | 2           | 25                 |
| B           | 3           | 40                 |

### Q9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```SQL
WITH table1 as (SELECT s.customer_id, s.product_id, SUM(price) amount_spent_per_item
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, s.product_id
ORDER BY customer_id),

table2 as (SELECT customer_id, case when product_id = 2 then amount_spent_per_item*10
		WHEN product_id = 3 THEN amount_spent_per_item*10
		WHEN product_id = 1 THEN amount_spent_per_item*20 ELSE amount_spent_per_item END AS points_table
FROM table1)

SELECT customer_id, SUM(points_table) points
FROM table2
GROUP BY customer_id
```
| customer_id | points |
|-------------|--------|
| A           | 860    |
| B           | 940    |
| C           | 360    |

### Q10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```SQL
WITH table1 as (SELECT s.customer_id, SUM(price)*20 total_price
FROM dannys_diner.sales s
JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
JOIN dannys_diner.menu menu
ON s.product_id = menu.product_id
WHERE EXTRACT(MONTH FROM order_date) = 1 AND join_date <= order_date
GROUP BY s.customer_id),

table2 as (SELECT s.customer_id, s.product_id, SUM(price) total_amount_spent2
FROM dannys_diner.sales s
JOIN dannys_diner.members m
ON s.customer_id = m.customer_id
JOIN dannys_diner.menu menu
ON s.product_id = menu.product_id
WHERE EXTRACT(MONTH FROM order_date) = 1 AND join_date > order_date
GROUP BY s.customer_id, s.product_id),

table3 AS (SELECT customer_id, 
     CASE WHEN product_id = 1 THEN total_amount_spent2 * 20
     WHEN product_id = 2 THEN total_amount_spent2*10      
     WHEN product_id = 3 THEN total_amount_spent2*10   
     ELSE total_amount_spent2 END AS new_point
     FROM table2)
    
SELECT t1.customer_id, (total_price + SUM(new_point)) total_points
FROM table1 t1
JOIN table3 t3
ON t1.customer_id = t3.customer_id
GROUP BY t1.customer_id , total_price
ORDER BY t1.customer_id
```
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 940          |











