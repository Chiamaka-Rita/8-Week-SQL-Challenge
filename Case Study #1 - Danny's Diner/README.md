## Problem Statement
Danny a restaurant owner want to use data to answer simple questions questions about his customers. The insights from these analysis will help him decide if he should expand his exisiting customer loyalty program. Also, he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

## Dataset
The tables were provided for this analysis and these are;
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















