**1. What is the total amount each customer spent at the restaurant?**

SELECT s.customer_id, sum(m.price) AS total_amount
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;

**2. How many days has each customer visited the restaurant?**

SELECT customer_id, count (DISTINCT order_date)
FROM dannys_diner.sales 
GROUP BY customer_id
ORDER BY customer_id;

**3. What was the first item from the menu purchased by each customer?**

WITH cte AS (SELECT s.customer_id, m.product_name, DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name, s.order_date) 
SELECT customer_id, product_name
FROM cte
WHERE rank = 1
GROUP BY customer_id, product_name;


**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

SELECT m.product_name, COUNT(s.product_id) as order_count
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY order_count DESC
LIMIT 1;


**5. Which item was the most popular for each customer?**

WITH cte AS (SELECT s.customer_id, m.product_name, DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY count(s.product_id) DESC) AS rank  
FROM dannys_diner.sales as s
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name)
SELECT 
customer_id, product_name
FROM cte
WHERE rank = 1

**6. Which item was purchased first by the customer after they became a member?**

WITH cte AS (SELECT e.customer_id, e.join_date, m.product_name, DENSE_RANK() OVER (PARTITION BY e.customer_id ORDER BY s.order_date) as rank
FROM dannys_diner.members as e
LEFT JOIN dannys_diner.sales as s 
ON s.customer_id = e.customer_id
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
WHERE s.order_date > e.join_date)
SELECT customer_id, product_name
FROM cte
WHERE rank = 1;


**7. Which item was purchased just before the customer became a member?**

WITH cte AS (SELECT e.customer_id, e.join_date, m.product_name, DENSE_RANK() OVER (PARTITION BY e.customer_id ORDER BY s.order_date DESC) rank
FROM dannys_diner.members as e
LEFT JOIN dannys_diner.sales as s 
ON s.customer_id = e.customer_id
JOIN dannys_diner.menu as m 
ON s.product_id = m.product_id
WHERE s.order_date < e.join_date)
SELECT customer_id, product_name
FROM cte
WHERE rank = 1;


**8. What is the total items and amount spent for each member before they became a member?**

SELECT s.customer_id, count(s.product_id) as total_items, sum(m.price) as amount_spent
FROM sales as s
JOIN members as e
ON s.customer_id = e.customer_id
JOIN menu as m
ON s.product_id = m.product_id
WHERE s.order_date < e.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;


**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

SELECT customer_id, SUM(CASE WHEN product_name = 'sushi' THEN price*20 
							 ELSE price*10
           					 END) AS points
FROM dannys_diner.menu AS m
JOIN dannys_diner.sales AS s 
ON m.product_id = s.product_id
GROUP BY customer_id
ORDER BY customer_id;


**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

WITH dates AS (SELECT e.customer_id, DATE(e.join_date+6) as end_date
FROM members as e)
SELECT s.customer_id, Sum(Case WHEN s.product_id = 1 THEN m.price*20
                          	   WHEN s.order_date BETWEEN e.join_date AND d.end_date THEN m.price*20
                               ELSE m.price*10
                          END) as points
                         
FROM dates as d
JOIN sales as s
ON s.customer_id = d.customer_id
JOIN members as e
ON s.customer_id = e.customer_id
JOIN menu as m
ON s.product_id = m.product_id
WHERE s.order_date < '2021-01-31'
GROUP BY s.customer_id;
