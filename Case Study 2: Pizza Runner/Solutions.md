**A. Pizza Metrics**

**1. How many pizzas were ordered?**

SELECT count(customer_id) as Pizza_order_count 
FROM customer_orders;

**2. How many unique customer orders were made?**

SELECT count(DISTINCT order_id) as Unique_orders 
FROM runner_orders;

**3. How many successful orders were delivered by each runner?**

SELECT runner_id, count(order_id)
FROM runner_orders1
WHERE Cancellation IS NULL
GROUP BY runner_id;

**4. How many of each type of pizza was delivered?**

SELECT c.pizza_id, count(c.pizza_id)
FROM customer_orders1 c
JOIN runner_orders1 r
ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.pizza_id;

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

SELECT c.customer_id, p.pizza_name, count(p.pizza_name)
FROM customer_orders1 c
JOIN pizza_names p
ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;

**6. What was the maximum number of pizzas delivered in a single order?**

SELECT c.order_id, count(c.pizza_id) as num_of_pizzas
FROM customer_orders1 c
GROUP BY c.order_id
ORDER BY num_of_pizzas DESC
LIMIT 1;

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

SELECT c.customer_id, 
	SUM(CASE WHEN (c.exclusions IS NOT NULL OR c.extras IS NOT NULL) THEN 1
    ELSE 0
    END) AS changes,
    SUM(CASE WHEN (c.exclusions IS NULL AND c.extras IS NULL) THEN 1
    ELSE 0
    END) AS no_changes
FROM customer_orders1 c
JOIN runner_orders1 r
ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id;

**8. How many pizzas were delivered that had both exclusions and extras?**

SELECT c.customer_id, 
	SUM(CASE WHEN (c.exclusions IS NOT NULL AND c.extras IS NOT NULL) THEN 1
    ELSE 0
    END) AS both
FROM customer_orders1 c
JOIN runner_orders1 r
ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id;

**9. What was the total volume of pizzas ordered for each hour of the day?**

SELECT 
    extract(hour from order_time) as hour,
	count(order_id) as num_of_pizzas
FROM customer_orders1
GROUP BY hour 
ORDER BY hour;


**10. What was the volume of orders for each day of the week?**

SELECT 
    to_char(order_time, 'day') as day,
	count(order_id) as num_of_pizzas
FROM customer_orders1
GROUP BY day 
ORDER BY day;


**B. Runner and Customer Experience**

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

SELECT EXTRACT(WEEK FROM registration_date) AS week,
count(runner_id) as num_of_runners
FROM runners
GROUP BY week
ORDER BY week;

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?**

SELECT
    r.runner_id,
    AVG(MINUTE(TIMEDIFF(r.pick_up_time, c.order_time))) AS time_mins
FROM cust_orders c
LEFT JOIN runner_orders_post r
	ON c.order_id = r.order_id
GROUP BY r.runner_id;

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

WITH order_cte AS
  (SELECT order_id,
          COUNT(order_id) AS pizzas_order_count,
          TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS prep_time
   FROM runner_orders1 as r
   JOIN customer_orders1 as c
   ON r.order_id = c.order_id
   WHERE cancellation IS NULL
   GROUP BY order_id)
SELECT pizzas_order_count,
       round(avg(prep_time), 2)
FROM order_cte;

**4. What was the average distance travelled for each customer?**

SELECT c.customer_id,
       round(avg(r.distance), 2) AS average_distance
FROM runner_orders1 as r
JOIN customer_orders1 as c
ON r.order_id = c.order_id
GROUP BY customer_id;

**5. What was the difference between the longest and shortest delivery times for all orders?**

SELECT MIN(duration) AS min_duration,
       MAX(duration) AS max_duration,
       MAX(duration) - MIN(duration) AS max_difference
FROM runner_orders1

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

SELECT runner_id,
       round(distance/duration, 2) AS average_speed
FROM runner_orders1
WHERE cancellation IS NULL
ORDER BY runner_id;

Trend- Yes, as the distance increases, the delivery time increases.

**7. What is the successful delivery percentage for each runner?**

SELECT runner_id,
       COUNT(pickup_time) AS delivered_orders,
       COUNT(order_id) AS total_orders,
       ROUND(100 * COUNT(pickup_time) / COUNT(order_id)) AS delivery_success_percentage
FROM runner_orders1
GROUP BY runner_id
ORDER BY runner_id;

