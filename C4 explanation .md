### 4.	Generate an order item for each record in the customers_orders table in the format of one of the following:
### •	Meat Lovers
### •	Meat Lovers - Exclude Beef
### •	Meat Lovers - Extra Bacon
### •	Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

Here are the tables that are relevent for this question:
##### customer_orders_v2
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/0385f514-73fa-4738-a4ba-8c66017bc486)


#### pizza_toppings
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/1b3e6f75-844c-4177-b40c-d296ef03878d)


Starting off I split the exclusions and extras columns from customer_orders_v2 into arrays so that I can actually work with them. A comma seperated list is useless to me. You'll notice that I also added an id column. The reason I add an id column is we have duplicate rows in the chart. Order_id 4, pizza_id 2 is literally the exact same row but we know that the customer was order two of the same pizzas on the same order. We need a way to differentiate the two (for the joins later) so the extra id column was needed. I call this CTE, ex (for EXtras and for EXclusions haha).  
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
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/a77810d3-acfa-4788-8e07-985738d317b6)

Next I needed to replace the topping_id in the table with the actual name of the topping. This was SO annoying to figure out. The only way to do this in bigquery (as far as I am aware) to is to 'flatten' the array, join the flattened array to pizza_toppings table on the pizza_id column and then unflatten the array. I had to use a correlated cross join to flatten the array (https://cloud.google.com/bigquery/docs/arrays#flattening_arrays). Unflattening the array wasn't complicated. It's just that you have to find the functions that do what you need it to do (aka scrolling through google's bigquery doc for an hour).  Anyway I started off by replacing the topping_ids with the actual names for the exclusions column first. Here is the code for all that below.
```sql
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
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/e273ae8b-2181-4b4e-ab20-840ecd508dca)

I also do the same thing for the extras columns in ex.

``` sql
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
```
![image](https://github.com/roodra01/Case-Study-2---Pizza-Runner/assets/129188359/e7df0415-7710-48b2-9ea1-0269f3b3d79b)


