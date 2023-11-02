# Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## Business Task
Danny is expanding his new Pizza Empire and at the same time, he wants to Uberize it, so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. 

## Entity Relationship Diagram

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

## Note to reader
If you want to get into the meat and potatoes of this project and *really* see me using SQL then skip ahead to question C4. That question was obnoxiously difficult to answer.

## Data Cleaning & Transformation
Before we get into the questions I wanna clean up the data a little so we don't run into problems later on. The challenge states that customer_orders and runner_orders has some problems. Lets dive into them to see what the issues are. 

### customer_orders
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/7a33c1ad-f1c1-4ce8-aab6-0d6057a40b09)

The customers_orders table has null/NaN values and blank values in the exclusions and extras. To solve this issue we will just create a table that replaces that empty/NaN values with Null so that the we have some consistency.
#### code:
```sql
CREATE TEMP TABLE customer_orders_v2 AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions='' or exclusions LIKE 'null' THEN NULL
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras ='' or extras LIKE 'null' THEN NULL
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;

SELECT * FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/c1d5daf4-c94a-4b56-984f-3cea2b4e01e4)


Fixed! This will be the table we use when dealing with questions that require the customer_orders.

### runner_orders
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ea5d6962-6464-45c5-8f78-da13d1d55143)

This one is a little bit icky but nothing we can't deal with. First, for the distance column some of the values has KM and others don't. For that issue we can just trim the KM off. Otherwise we'll replace the empty values with null. Next the duration column has minutes/min which we will also trim off and replace empty values with null. For the cancellation column we have to change empty values to null. One other issue here is data types. Lets take a look at the data types in this table:
#### Code:
```sql
SELECT column_name,data_type
FROM pizza_runner.INFORMATION_SCHEMA.COLUMNS
WHERE table_name= 'runner_orders'
```

![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/bc8d9e83-3787-42e3-8f8f-94068f83bb83)

Ideally we want our pickup time to be DATETIME, and our distance/duration to FLOAT64 so we actually work with these values in SQL. So let's also make that happen.

#### Code:
``` sql
CREATE TEMP TABLE runner_orders_v2 AS
SELECT 
  order_id, 
  runner_id,  
	SAFE_CAST(CASE
	  WHEN pickup_time LIKE 'null' THEN NULL
	  ELSE pickup_time
	  END as DATETIME) as pickup_time,
  SAFE_CAST(CASE
	  WHEN distance LIKE 'null' THEN NULL
	  WHEN distance LIKE '%km' THEN TRIM(distance,'km')
	  ELSE distance 
    END AS FLOAT64) as distance,
  SAFE_CAST(CASE
	  WHEN duration LIKE 'null' THEN NULL
	  WHEN duration LIKE '%mins' THEN TRIM(duration, 'mins')
	  WHEN duration LIKE '%minute' THEN TRIM(duration, 'minute') 
	  WHEN duration LIKE '%minutes' THEN TRIM(duration, 'minutes')
	  ELSE duration
	  END AS FLOAT64) as duration,
  CASE
	  WHEN cancellation='' or cancellation LIKE 'null' THEN NULL
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders; 

SELECT * FROM runner_orders_v2 ORDER BY order_id
```

![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ccd163ea-201a-432f-9013-2822eb368cf4)

Fixed! This will be the table we use when dealing with questions that require the runner_orders.

## Solutions
### A. Pizza Metrics
#### 1. How many pizzas were ordered?
```sql
SELECT count(*) as pizza_count FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/0abdb6d5-95c3-4e17-9808-d88f7d787f82)

#### 2. How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT(order_id)) as unique_orders FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/effb60ed-7b73-44a8-bba3-6e80a7e2de74)

#### 3. How many successful orders were delivered by each runner?

```sql
SELECT runner_id,
count(*) as successful_orders
FROM runner_orders_v2
WHERE cancellation is NULL
GROUP BY runner_id
ORDER BY runner_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/c93ef2a2-b8ee-4043-b37c-23b23425306c)


#### 4. How many of each type of pizza was delivered?

```sql

SELECT pizza_id, count(*) as successful_pizza_count
FROM runner_orders_v2 as run
JOIN customer_orders_v2 as cus
ON run.order_id=cus.order_id
WHERE cancellation IS NULL
GROUP BY pizza_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/5c7c1a2f-9a83-4cf6-baa9-d2989a215f72)


#### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT 
  c.customer_id, 
  p.pizza_name, 
  COUNT(p.pizza_name) AS order_count
FROM pizza_runner.customer_orders as c
JOIN pizza_runner.pizza_names AS p
  ON c.pizza_id= p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;

```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/db21a22b-08fd-4722-a0d8-2fbaabf2f86a)

#### 6. What was the maximum number of pizzas delivered in a single order?
```sql
WITH pizzas_per_ord AS (
	SELECT 
	order_id, 
	COUNT(*) as pizza_count
	FROM customer_orders_v2
	GROUP BY order_id)

SELECT MAX(pizza_count) as max_ordered FROM pizzas_per_ord
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/d5fca758-2687-4d8a-bac0-4f518b827922)

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
  cus.customer_id,
  SUM(
    CASE WHEN cus.exclusions IS NOT NULL OR cus.extras IS NOT NULL THEN 1
    ELSE 0
    END) AS at_least_1_change,
  SUM(
    CASE WHEN cus.exclusions IS NULL AND cus.extras IS NULL THEN 1 
    ELSE 0
    END) AS no_change
FROM customer_orders_v2 AS cus
JOIN runner_orders_v2 AS run
  ON cus.order_id = run.order_id
WHERE run.distance IS NOT NULL
GROUP BY cus.customer_id
ORDER BY cus.customer_id;

```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/f2a02d39-e073-4d13-8fc5-ae4b7f292ba7)

Note: I add WHERE run.distance IS NOT NULL because we are the question is how many _DELIVERED_ pizzas. run.distance not being null ensures the pizza was delivered.

#### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT  
  SUM(
    CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1
    ELSE 0
    END) AS pizza_w_ex_ex
FROM customer_orders_v2 AS cus
JOIN runner_orders_v2 AS run
  ON cus.order_id = run.order_id
WHERE run.distance IS NOT NULL
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ff782063-7d4f-4468-b9e2-d19e83658235)


#### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
  EXTRACT(HOUR FROM order_time) AS hour_of_day, 
  COUNT(order_id) AS pizza_count
FROM customer_orders_v2
GROUP BY EXTRACT(HOUR FROM order_time)
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/aa1bcf19-7e47-4898-ad2e-20570232eac9)

Note: The hours are in military time.

#### 10. What was the volume of orders for each day of the week?
```sql
SELECT 
  EXTRACT(DAYOFWEEK FROM order_time) AS day_of_week, 
  COUNT(order_id) AS pizza_count
FROM customer_orders_v2
GROUP BY EXTRACT(DAYOFWEEK FROM order_time)
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/9facad12-5a66-4815-8cab-98afb76ce007)

Note: The numbers under day_of_week correspond to a day of the week starting with Sunday as 1 to Saturday as 7. If you wanted a chart with the Day of the week as the name it'd be simple to do it. You would just use CASE to add a column to correspond each number with a day of the week.

### B. Runner and Customer Experience
#### 1.	How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql

SELECT 
  EXTRACT(WEEK FROM registration_date) AS week, 
  COUNT(order_id) AS runner_counts
FROM customer_orders_v2
GROUP BY EXTRACT(WEEK FROM registration_date);
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/06c61400-a15c-487f-a561-c50856f2997e)

#### 2.	What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH time_per_order AS(
SELECT 
run.order_id,
order_time,
pickup_time,
DATETIME_DIFF(pickup_time, SAFE_CAST(order_time as DATETIME),  MINUTE) as time_to_pickup
FROM runner_orders_v2 AS run
JOIN customer_orders_v2 AS cus
ON run.order_id=cus.order_id
WHERE pickup_time IS NOT NULL
GROUP BY run.order_id, order_time, pickup_time
ORDER BY run.order_id)

SELECT avg(time_to_pickup) AS avg_time_to_pickup
FROM time_per_order;
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/27e973f3-4b97-414a-93ce-a9672c8b5d05)


#### 3.	Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH pizza_numberVStime AS (
SELECT 
count(*) as number_of_pizzas,
DATETIME_DIFF(pickup_time, SAFE_CAST(order_time as DATETIME),  MINUTE) as time_to_pickup
FROM runner_orders_v2 AS run
JOIN customer_orders_v2 AS cus
ON run.order_id=cus.order_id
WHERE pickup_time IS NOT NULL
GROUP BY run.order_id, order_time, pickup_time
ORDER BY run.order_id)

SELECT
number_of_pizzas,
avg(time_to_pickup) as avg_time
FROM pizza_numberVStime
GROUP BY number_of_pizzas
ORDER BY number_of_pizzas

```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/fc24eeed-8e9a-4a03-bfe7-d190cd70f4b1)

Prep time increases with the amount of pizzas ordered.

#### 4.	What was the average distance travelled for each customer?
```sql

WITH order_distance AS (SELECT 
customer_id, run.order_id, avg(distance) as distance
FROM runner_orders_v2 AS run
JOIN customer_orders_v2 AS cus
ON run.order_id=cus.order_id
WHERE pickup_time IS NOT NULL
GROUP BY customer_id, run.order_id)

SELECT customer_id, avg(distance) as avg_distance
FROM order_distance
GROUP BY customer_id
ORDER BY customer_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/9fbdaf0c-2218-477a-9e47-5b9e016d0d78)


#### 5.	What was the difference between the longest and shortest delivery times for all orders?
```sql
WITH order_duration AS(
SELECT 
customer_id, run.order_id, avg(duration) as duration
FROM runner_orders_v2 AS run
JOIN customer_orders_v2 AS cus
ON run.order_id=cus.order_id
WHERE pickup_time IS NOT NULL
GROUP BY customer_id, run.order_id)

SELECT max(duration) - min(duration) as max_difference
FROM order_duration
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/e595e9cb-8a24-4e9a-94dc-bf7f147c82ec)

#### 6.	What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT runner_id, order_id, round(distance/(duration/60), 3) as avg_speed
FROM runner_orders_v2
WHERE distance IS NOT NULL
ORDER BY runner_id, order_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ebbe53f5-cf30-4c73-aea2-23f2b371d1bd)

#### 7.	What is the successful delivery percentage for each runner?
```sql
SELECT 
runner_id, 
ROUND(100 * SUM(
  CASE WHEN distance IS NULL THEN 0
  ELSE 1 END) / COUNT(*), 0) AS success_perc
FROM runner_orders_v2
GROUP BY runner_id
ORDER BY runner_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/92d3299f-e019-4696-b932-0811c59e6830)

### C. Ingredient Optimisation
#### 1.	What are the standard ingredients for each pizza?
```sql
WITH seperated_tops AS (
SELECT
pizza_id,
split(toppings) as toppings
FROM `pizza_runner.pizza_recipes`),

sep_pizza_tops AS (
SELECT 
pizza_id, 
cast(split_toppings as INT64) as split_toppings
FROM seperated_tops 
CROSS JOIN UNNEST(seperated_tops.toppings) as split_toppings)

SELECT 
pizza_id,
STRING_AGG(topping_name, ", ") as topping_name
FROM 
sep_pizza_tops as s
JOIN `pizza_runner.pizza_toppings` as t
on s.split_toppings= t.topping_id 
GROUP BY pizza_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/01af1f15-00d8-48d6-9e5a-434085fcfbcd)

#### 2.	What was the most commonly added extra?
```sql
WITH split_extras AS( 
	SELECT 
order_id,
split(extras) as extras
FROM
customer_orders_v2
WHERE extras IS NOT NULL)

SELECT
topping_name,
number_of_times_ordered
FROM `pizza_runner.pizza_toppings` as t
JOIN(
	SELECT 
		CAST(split_extras2 as INT64) as extras,
		count(split_extras2) as number_of_times_ordered
	FROM
	split_extras
	CROSS JOIN UNNEST(split_extras.extras) as split_extras2
GROUP BY split_extras2) as s
ON s.extras= t.topping_id 

```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/a7978bb2-88a0-4584-9009-bb337fb67ec3)


#### 3.	What was the most common exclusion?
```sql
WITH split_exclusions AS( 
	SELECT 
order_id,
split(exclusions) as exclusions
FROM
customer_orders_v2
WHERE exclusions IS NOT NULL)

SELECT
topping_name,
number_of_times_ordered
FROM `pizza_runner.pizza_toppings` as t
JOIN(
	SELECT 
		CAST(split_exclusions2 as INT64) as exclusions,
		count(split_exclusions2) as number_of_times_ordered
	FROM
	split_exclusions
	CROSS JOIN UNNEST( split_exclusions.exclusions) as  split_exclusions2
GROUP BY split_exclusions2) as s
ON s.exclusions= t.topping_id 
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/05385d4d-b7a0-426e-8335-209d7de83402)


#### 4.	Generate an order item for each record in the customers_orders table in the format of one of the following:
#### •	Meat Lovers
#### •	Meat Lovers - Exclude Beef
#### •	Meat Lovers - Extra Bacon
#### •	Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```sql
WITH ex AS (
SELECT 
ROW_NUMBER() OVER (ORDER BY order_id, pizza_id) as id,
order_id,
customer_id,
pizza_id,
order_time,
split(exclusions) as exclusions,
split(extras) as extras
FROM
customer_orders_v2
),

spl_exclusions as 
(SELECT 
	order_id, pizza_id, cast(split_exclusions2 AS int64) as sp
	FROM
	ex
	CROSS JOIN UNNEST( ex.exclusions) as  split_exclusions2
),

named_exc AS 
(SELECT
	order_id, pizza_id, sp, topping_name
FROM spl_exclusions as s
JOIN `pizza_runner.pizza_toppings` as p
ON s.sp=p.topping_id),


array_name_exc AS
(SELECT order_id, pizza_id, array_agg(topping_name) as topping_name
FROM named_exc
GROUP BY order_id, pizza_id),

spl_extras as 
(SELECT 
	order_id, pizza_id, cast(split_extras2 AS int64) as sp
	FROM
	ex
	CROSS JOIN UNNEST( ex.extras) as  split_extras2
),

named_ext AS 
(SELECT
	order_id, pizza_id, sp, topping_name
FROM spl_extras as s
JOIN `pizza_runner.pizza_toppings` as p
ON s.sp=p.topping_id),


array_name_ext AS
(SELECT order_id, pizza_id, array_agg(topping_name) as topping_name
FROM named_ext
GROUP BY order_id, pizza_id),

almost_extras AS
(SELECT 
ROW_NUMBER() OVER (ORDER BY ex.order_id, ex.pizza_id) as id,
 ex.order_id,
 ex.pizza_id,
 unspex,
CASE
	WHEN ex.pizza_id= 1 THEN 'Meatlovers'
	WHEN ex.pizza_id= 2 THEN 'Vegetarian'
END,
 CASE 
 	WHEN ARRAY_LENGTH(extras) > 1 THEN CONCAT(' - Extra ' , ARRAY_TO_STRING(topping_name, ', ') )
	ELSE CONCAT(' - Extra ' , topping_name[safe_offset(0)])
END as extras
FROM ex
FULL OUTER JOIN array_name_ext as aext
ON ex.order_id=aext.order_id AND ex.pizza_id=aext.pizza_id
ORDER BY order_id, pizza_id),

final_list as (SELECT 
 ex.order_id,
 ex.pizza_id,
CASE
	WHEN ex.pizza_id= 1 THEN 'Meatlovers'
	WHEN ex.pizza_id= 2 THEN 'Vegetarian'
END as pizza,
CASE 
WHEN ARRAY_LENGTH(exclusions) > 1 THEN CONCAT(' - Exclude ' , ARRAY_TO_STRING(topping_name, ', ') ) 
ELSE CONCAT(' - Exclude ' , topping_name[safe_offset(0)])
END as exclusions,
 almost_extras.extras
FROM ex
JOIN almost_extras
ON ex.id=almost_extras.id
FULL OUTER JOIN array_name_exc as aex
ON ex.order_id=aex.order_id AND ex.pizza_id=aex.pizza_id
ORDER BY order_id, pizza_id)

SELECT 
order_id,
pizza_id,
CASE
	WHEN exclusions is NULL and EXTRAS is NOT NULL THEN CONCAT(pizza, extras)
	WHEN extras is NULL and exclusions is NOT NULL THEN CONCAT(pizza, exclusions)
	WHEN extras is NULL AND exclusions is NULL THEN pizza 
	ELSE CONCAT(pizza, exclusions, extras)
	END as order_item
FROM final_list

```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/4ebb8f5e-727c-4189-80b9-95ea2cceffe7)



### D.  Pricing and Ratings
#### 1.	If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
WITH uncancelled_orders AS (
SELECT
*
FROM  customer_orders_v2
WHERE EXISTS( 
	SELECT *
	FROM runner_orders_v2
	WHERE customer_orders_v2.order_id=runner_orders_v2.order_id AND cancellation IS NULL
))

SELECT
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 12
      WHEN pizza_id = 2 THEN 10
      END )
       
FROM uncancelled_orders


```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/b29e9ebf-8ac9-4771-a81d-6225ece55ae8)

#### 2.	What if there was an additional $1 charge for any pizza extras?

```sql
WITH uncancelled_orders AS (
SELECT
pizza_id,
	split(extras) as extras
FROM  customer_orders_v2
WHERE EXISTS( 
	SELECT 
	*
	FROM runner_orders_v2
	WHERE customer_orders_v2.order_id=runner_orders_v2.order_id AND cancellation IS NULL
))


SELECT
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 12  
      WHEN pizza_id = 2 THEN 10  
      END )  + SUM(ARRAY_LENGTH(extras)) as cost
   
     
FROM uncancelled_orders
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/bd415493-0310-4141-a962-1e64ef2456f4)

#### •	Add cheese is $1 extra
```sql
WITH uncancelled_orders AS (
SELECT
pizza_id,
	split(extras) as extras
FROM  customer_orders_v2
WHERE EXISTS( 
	SELECT 
	*
	FROM runner_orders_v2
	WHERE customer_orders_v2.order_id=runner_orders_v2.order_id AND cancellation IS NULL
))




SELECT
  SUM(
    CASE 
      WHEN pizza_id = 1 THEN 12  
      WHEN pizza_id = 2 THEN 10  
      END )  + 
	 SUM(ARRAY_LENGTH(extras))  +
	 	SUM(
		 CASE 
			WHEN ' 4' IN unnest(extras) THEN 1
			ELSE 0
			END) as cost_with_cheese
FROM uncancelled_orders
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/0768dca2-be33-4ab7-abf7-a0fdedd0c31b)
#### 3.	The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql

SELECT distinct(order_id),
CASE
WHEN 1=1 THEN  FLOOR(1 + 5* RAND()) 
END
AS rating
FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/06d2d2b5-4a0e-4d63-90e6-5104cef41ff4)

#### 4.	Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
#### •	customer_id
#### •	order_id
#### •	runner_id
#### •	rating
#### •	order_time
#### •	pickup_time
#### •	Time between order and pickup
#### •	Delivery duration
#### •	Average speed
#### •	Total number of pizzas
```sql
SELECT
cus.order_id,
runner_id,
CASE
	WHEN 1=1 THEN  FLOOR(1 + 5* RAND()) 
  END as rating,
order_time,
pickup_time,
DATETIME_DIFF(pickup_time, SAFE_CAST(order_time as DATETIME),  MINUTE) as time_to_pickup,
round(distance/(duration/60), 3) as speed,
count(*) as pizza_count
FROM runner_orders_v2 AS run
JOIN customer_orders_v2 AS cus
ON run.order_id=cus.order_id
WHERE pickup_time IS NOT NULL 
GROUP BY cus.order_id, runner_id, distance, duration,pickup_time, order_time
ORDER BY cus.order_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/7744e749-7ca9-429e-9ca7-d02ea40ecee3)

Note: I use random numbers for the ratings here but you could easily put into ratings with a CASE WHEN depending on order_id.

#### 5.	If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
WITH uncancelled_orders AS (
SELECT
*
FROM  customer_orders_v2
WHERE EXISTS( 
	SELECT *
	FROM runner_orders_v2
	WHERE customer_orders_v2.order_id=runner_orders_v2.order_id AND cancellation IS NULL
)),

pizza_money AS (
SELECT
  SUM(
    CASE
      WHEN pizza_id = 1 THEN 12
      WHEN pizza_id = 2 THEN 10
      END )
       
FROM uncancelled_orders)

SELECT (SELECT * FROM pizza_money) - (SELECT sum(distance)*.3 FROM runner_orders_v2) AS left_over_money
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/6474979e-84e8-43ef-b5c0-642484c74faf)
