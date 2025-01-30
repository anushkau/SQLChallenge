DROP TABLE IF EXISTS customer_orders1;

CREATE TEMPORARY TABLE customer_orders1 AS
SELECT order_id,
       customer_id,
       pizza_id,
       CASE
           WHEN exclusions = '' THEN NULL
           WHEN exclusions = 'null' THEN NULL
           ELSE exclusions
       END AS exclusions,
       CASE
           WHEN extras = '' THEN NULL
           WHEN extras = 'null' THEN NULL
           ELSE extras
       END AS extras,
       order_time
FROM customer_orders;

SELECT * FROM customer_orders1;

DROP TABLE IF EXISTS runner_orders1;

CREATE TEMPORARY TABLE runner_orders1 AS

SELECT order_id,
       runner_id,
       CASE
           WHEN pickup_time LIKE 'null' THEN NULL
           ELSE pickup_time
       END AS pickup_time,
       CASE
           WHEN distance LIKE 'null' THEN NULL
           WHEN distance LIKE '%km' THEN TRIM('km' from distance)
       END AS distance,
       CASE
           WHEN duration LIKE 'null' THEN NULL
           WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  	   WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  	   WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
       END AS duration,
       CASE
           WHEN cancellation LIKE '' THEN NULL
           WHEN cancellation LIKE 'null' THEN NULL
           ELSE cancellation
       END AS cancellation
FROM runner_orders;

SELECT * FROM runner_orders1;
