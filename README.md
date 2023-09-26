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

The customers_orders table has null/NaN values and blank values in the exclusions and extras. To solve this issue we will just create a table that replaces that the Null/NaN values with ' ' so that the we have some consistency.
#### code:
```sql
CREATE TEMP TABLE customer_orders_v2 AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;

SELECT * FROM customer_orders_v2
````
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ed794b82-12d1-4c2f-8929-4c730b0e18c4)

Fixed! This will be the table we use when dealing with questions that require the customer_orders.

### runner_orders
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/ea5d6962-6464-45c5-8f78-da13d1d55143)

This one is a little bit icky but nothing we can't deal with. First, for the distance column some of the values has KM and others don't. For that issue we can just trim the KM off. Otherwise we'll replace the null values with ' '. Next the duration column has distance/ dis
