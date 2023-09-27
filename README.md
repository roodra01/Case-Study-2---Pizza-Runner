# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- Solution
  - [Data Cleaning and Transformation](#-data-cleaning--transformation)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)
 
## Business Task
Danny is expanding his new Pizza Empire and at the same time, he wants to Uberize it, so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. 

## Entity Relationship Diagram

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

## 🧼 Data Cleaning & Transformation
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
### 1. How many pizzas were ordered?
```sql
SELECT count(*) as pizza_count FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/0abdb6d5-95c3-4e17-9808-d88f7d787f82)

### 2. How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT(order_id)) as unique_orders FROM customer_orders_v2
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/effb60ed-7b73-44a8-bba3-6e80a7e2de74)

### 3. How many successful orders were delivered by each runner?

```sql
SELECT runner_id,
count(*) as successful_orders
FROM runner_orders_v2
WHERE cancellation is NULL
GROUP BY runner_id
ORDER BY runner_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/c93ef2a2-b8ee-4043-b37c-23b23425306c)


### 4. How many of each type of pizza was delivered?

```sql

SELECT pizza_id, count(*) as successful_pizza_count
FROM runner_orders_v2 as run
JOIN customer_orders_v2 as cus
ON run.order_id=cus.order_id
WHERE cancellation IS NULL
GROUP BY pizza_id
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/5c7c1a2f-9a83-4cf6-baa9-d2989a215f72)


### 5. How many Vegetarian and Meatlovers were ordered by each customer?
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

### 6. What was the maximum number of pizzas delivered in a single order?
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

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
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

### 8. How many pizzas were delivered that had both exclusions and extras?
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


### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
  EXTRACT(HOUR FROM order_time) AS hour_of_day, 
  COUNT(order_id) AS pizza_count
FROM customer_orders_v2
GROUP BY EXTRACT(HOUR FROM order_time)
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/aa1bcf19-7e47-4898-ad2e-20570232eac9)

Note: The hours are in military time.

### 10. What was the volume of orders for each day of the week?
```sql
SELECT 
  EXTRACT(DAYOFWEEK FROM order_time) AS day_of_week, 
  COUNT(order_id) AS pizza_count
FROM customer_orders_v2
GROUP BY EXTRACT(DAYOFWEEK FROM order_time)
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/9facad12-5a66-4815-8cab-98afb76ce007)

Note: The numbers under day_of_week correspond to a day of the week starting with Sunday as 1 to Saturday as 7. If you wanted a chart with the Day of the week as the name it'd be simple to do it. You would just use CASE to add a column to correspond each number with a day of the week.

