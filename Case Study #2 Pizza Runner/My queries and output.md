```mysql
use pizza_runner;
```
### A. Pizza Metrics

#### How many pizzas were ordered?
```mysql
select count(c.order_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
```
#### output
![Screen Shot 2022-09-05 at 2 56 47 AM](https://user-images.githubusercontent.com/92205707/188422520-31a594c4-fc79-4918-bc7b-0bb3449a40c1.png)

#### How many unique customer orders were made?
```mysql
select count(distinct c.order_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
```
#### output
![Screen Shot 2022-09-05 at 2 57 24 AM](https://user-images.githubusercontent.com/92205707/188422631-3c579470-b1d8-404b-bea9-6620e4cc4923.png)

#### How many successful orders were delivered by each runner?
```mysql
select r.runner_id, count(distinct c.order_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
where r.distance != 'null' 
group by r.runner_id
order by r.runner_id asc
```
#### output
![Screen Shot 2022-09-05 at 2 58 04 AM](https://user-images.githubusercontent.com/92205707/188422750-73867678-fc30-4e57-aafa-c1e1dee81f7d.png)

#### How many of each type of pizza was delivered?
```mysql
select p.pizza_name, count(c.order_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
where r.distance != 'null'
group by p.pizza_name
```
#### output
![Screen Shot 2022-09-05 at 2 58 39 AM](https://user-images.githubusercontent.com/92205707/188422859-d84b2aea-586d-46d9-ae32-f3e9b7f255a5.png)

#### How many Vegetarian and Meatlovers were ordered by each customer?
```mysql
select c.customer_id, p.pizza_name, count(c.order_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
group by c.customer_id, p.pizza_name
```
#### output
![Screen Shot 2022-09-05 at 2 59 16 AM](https://user-images.githubusercontent.com/92205707/188422986-d35fd535-f176-41a5-a966-d27d70820e16.png)

#### What was the maximum number of pizzas delivered in a single order?
```mysql
select r.order_id, count(c.order_id) as count_orders from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
where r.distance != 'null'
group by r.order_id
order by count(c.order_id) desc
```
#### output
![Screen Shot 2022-09-05 at 2 59 52 AM](https://user-images.githubusercontent.com/92205707/188423108-835135e1-5d90-41b6-a8a4-86527e2a2982.png)

#### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```mysql
with cte as (select c.customer_id, count(c.exclusions) as count_changes from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
where c.exclusions != 'null' and r.distance != 'null' or c.extras != 'null' and r.distance != 'null'
group by c.customer_id) select c.customer_id, count(c.pizza_id) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
where customer_id not in (select customer_id from cte) and r.distance != 'null'
group by c.customer_id
```
#### output (at least 1 change)
![Screen Shot 2022-09-05 at 3 02 42 AM](https://user-images.githubusercontent.com/92205707/188423635-f09dc4ce-77eb-47d9-a8f9-cbb41c9a005f.png)
#### output (no changes)
![Screen Shot 2022-09-05 at 3 03 16 AM](https://user-images.githubusercontent.com/92205707/188423711-dcc4f5bf-7587-4a0f-8e73-6937f1988e5f.png)

#### How many pizzas were delivered that had both exclusions and extras?
```mysql
select c.customer_id, count(c.exclusions + c.extras) from customer_orders as c
join runner_orders as r on c.order_id = r.order_id
join pizza_names as p on p.pizza_id = c.pizza_id
where c.exclusions != 'null' and r.distance != 'null' and c.extras != 'null' and r.distance != 'null'
group by c.customer_id;
```
#### output
![Screen Shot 2022-09-05 at 3 04 00 AM](https://user-images.githubusercontent.com/92205707/188423871-cbc2e412-725e-4b17-a939-9c2d4d9f813f.png)

#### What was the total volume of pizzas ordered for each hour of the day? (done in tableau)
#### output
![Screen Shot 2022-09-05 at 3 05 40 AM](https://user-images.githubusercontent.com/92205707/188424157-102ac672-36ec-44b0-8048-545c37157980.png)

#### What was the volume of orders for each day of the week? (done in tableau)
#### output
![Screen Shot 2022-09-05 at 3 06 12 AM](https://user-images.githubusercontent.com/92205707/188424243-b1728ceb-3d74-4256-81d9-6880e5fa4354.png)

### B. Runner and Customer Experience

#### How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01) (done in Tableau)
#### output
![Screen Shot 2022-09-05 at 3 06 50 AM](https://user-images.githubusercontent.com/92205707/188424414-e3b1998e-d4f8-4eb7-976d-82c4cc4d0625.png)

#### What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order? (done in Tableau)
#### output
![Screen Shot 2022-09-05 at 3 07 30 AM](https://user-images.githubusercontent.com/92205707/188424552-47ea1bd6-57e2-4e7b-8aa3-e352f075871c.png)

#### Is there any relationship between the number of pizzas and how long the order takes to prepare? (done in Tableau)
#### output
![Screen Shot 2022-09-05 at 3 08 09 AM](https://user-images.githubusercontent.com/92205707/188424689-9e723d33-7ed9-4bea-be65-3e0c7dee8a80.png)

#### What was the average distance travelled for each customer?
```mysql
with cte as (select c.customer_id, c.order_id, r.pickup_time, avg(r.distance) as avg_distance from customer_orders as c
join runner_orders as r on r.order_id = c.order_id
where r.distance != 'null'
group by c.customer_id, c.order_id, r.pickup_time
order by c.customer_id asc) 
select customer_id, avg(avg_distance)
from cte
group by customer_id
```
#### output
![Screen Shot 2022-09-05 at 3 12 11 AM](https://user-images.githubusercontent.com/92205707/188425429-41c91f11-8d87-4cb5-bf8a-639541166a9c.png)

#### What was the difference between the longest and shortest delivery times for all orders?
```mysql
select max(duration) - min(duration) from runner_orders
where distance != 'null'
```
#### output
![Screen Shot 2022-09-05 at 3 12 44 AM](https://user-images.githubusercontent.com/92205707/188425532-8693d470-d758-4bd6-af5a-394b5b6a8009.png)

#### What was the average speed for each runner for each delivery and do you notice any trend for these values?
```mysql
select r.runner_id, r.order_id, r.pickup_time, count(c.order_id), r.distance/duration * 60 as avg_speed
from runner_orders as r
join customer_orders as c on r.order_id = c.order_id
where distance != 'null'
group by r.runner_id, r.order_id, r.distance/duration * 60, r.pickup_time
order by runner_id asc
```
#### output
![Screen Shot 2022-09-05 at 3 13 25 AM](https://user-images.githubusercontent.com/92205707/188425694-ba86a783-f335-4dd1-9475-57b66d2543de.png)

#### What is the successful delivery percentage for each runner?
```mysql
create view total_orders as select runner_id, count(duration) as total_orders
from runner_orders
group by runner_id;

select runner_orders.runner_id, round(count(runner_orders.duration)/total_orders * 100,1) as successful_orders
from runner_orders
join total_orders on total_orders.runner_id = runner_orders.runner_id
where duration != 'null'
group by runner_id
```
#### output
![Screen Shot 2022-09-05 at 3 14 10 AM](https://user-images.githubusercontent.com/92205707/188425834-3f44d08a-be0f-4e60-bca6-d3f756a9d94e.png)

### C. Ingredient Optimisation

#### What are the standard ingredients for each pizza?
```mysql
select n.pizza_name,
(case
when r.toppings = '1, 2, 3, 4, 5, 6, 8, 10' then 'Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Peppers' 
when r.toppings = '4, 6, 7, 9, 11, 12' then 'Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce' end) as toppings_name
from pizza_recipes as r 
join pizza_names as n on n.pizza_id = r.pizza_id
```
#### output
![Screen Shot 2022-09-05 at 3 14 59 AM](https://user-images.githubusercontent.com/92205707/188425992-b2f12a32-30eb-489d-a0a2-aeab1dde1949.png)

#### What was the most commonly added extra?
```mysql
with cte as (select c.order_id, c.customer_id, c.extras, (case when c.extras = '1' then 'Bacon' 
when c.extras = '1, 4' then 'Bacon, Cheese'
when c.extras = '1, 5' then 'Bacon, Chicken' end) as extras_name
from pizza_recipes as r 
join pizza_names as n on n.pizza_id = r.pizza_id
join customer_orders as c on r.pizza_id = r.pizza_id
where c.extras != 'null')
select extras_name, round(count(extras_name)/2,0) from cte group by extras_name
```
#### output
![Screen Shot 2022-09-05 at 3 15 34 AM](https://user-images.githubusercontent.com/92205707/188426087-312630aa-61b4-409a-8676-d1207fc4fefb.png)

#### What was the most common exclusion?
```mysql
with cte as (select c.order_id, c.order_time, c.pizza_id, c.customer_id, c.exclusions,
(case when c.exclusions = '4' then 'Cheese' when c.exclusions = '2, 6' then 'BBQ Sauce, Mushrooms' end) as exclusions_name
from pizza_recipes as r 
join pizza_names as n on n.pizza_id = r.pizza_id
join customer_orders as c on r.pizza_id = r.pizza_id
where c.exclusions != 'null')
select exclusions_name, round(count(exclusions_name)/2,0) from cte group by exclusions_name
```
#### output
![Screen Shot 2022-09-05 at 3 16 27 AM](https://user-images.githubusercontent.com/92205707/188426267-c0fe97e2-fa19-4a3c-bca4-3465188a1e3e.png)

#### Generate an order item for each record in the customers_orders table in the format of one of the following:
-- 1. Meat Lovers
-- 2. Meat Lovers - Exclude Beef
-- 3. Meat Lovers - Extra Bacon
-- 4. Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```mysql
with cte as (select c.pizza_id, c.customer_id, c.order_id, (case when c.pizza_id = '1' then 'Meat Lovers'
when c.pizza_id = '2' then 'Vegan lovers' end) as pizza_name, 
(case when c.pizza_id = '1' and c.extras = 'null' then 'Meat Lovers'
when c.pizza_id = '2' and c.extras = 'null' or c.extras = null then 'Vegan lovers'
when c.extras = '1' or c.extras = '1, 5' or c.extras = '1, 4' and c.pizza_id = 2 then 'Vegan plus Meat Lovers - Extra Bacon'
when c.extras = '1' or c.extras = '1, 5' or c.extras = '1, 4' and c.pizza_id = 1 then 'Meat Lovers - Extra Bacon' end) as order_item
from pizza_recipes as r 
join pizza_names as n on n.pizza_id = r.pizza_id
join customer_orders as c on r.pizza_id = r.pizza_id)
select order_id, order_item, round(count(order_id)/2,0) as num_of_pizza_per_order_with_extra_bacon_or_not from cte 
group by order_id, order_item
```
#### output
![Screen Shot 2022-09-05 at 3 17 16 AM](https://user-images.githubusercontent.com/92205707/188426418-c08be2e4-22d7-4eee-9413-c76cac56f91f.png)

#### Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
#### 1. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```mysql
select c.order_id, c.pizza_id, p.pizza_name,
(case when c.exclusions = 'null' and c.extras = 'null' and c.pizza_id = 1 then 'Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = 'null' and c.extras = 'null' and c.pizza_id = 2 then 'Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce'
when c.exclusions = '4' and c.extras = '1, 5' and c.pizza_id = 1 then '2xBacon, BBQ Sauce, Beef, 2xChicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = '2, 6' and c.extras = '1, 4' and c.pizza_id = 1 then '2xBacon, Beef, 2xCheese, Chicken, Pepperoni, Salami' 
when c.exclusions = '4' and c.pizza_id = 1 then 'Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = '4' and c.pizza_id = 2 then 'Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce'
when c.extras = '1' and c.pizza_id = 1 then '2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami'
when c.extras = '1' and c.pizza_id = 2 then 'xBacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce' end) as ingredient
from customer_orders as c
join pizza_names as p on p.pizza_id = c.pizza_id
order by c.order_id asc
```
#### output
![Screen Shot 2022-09-05 at 3 18 06 AM](https://user-images.githubusercontent.com/92205707/188426576-d2aa6693-80f1-4877-90cd-1a1952f0c2ca.png)

#### What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first? (after I transformed data in SQL, I pivoted the table in Tableau)
```mysql
with cte as (select c.order_id, c.pizza_id, p.pizza_name,
(case when c.exclusions = 'null' and c.extras = 'null' and c.pizza_id = 1 then 'Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = 'null' and c.extras = 'null' and c.pizza_id = 2 then 'Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce'
when c.exclusions = '4' and c.extras = '1, 5' and c.pizza_id = 1 then 'Bacon, Bacon, BBQ Sauce, Beef, Chicken, Chicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = '2, 6' and c.extras = '1, 4' and c.pizza_id = 1 then 'Bacon, Bacon, Beef, Cheese, Cheese, Chicken, Pepperoni, Salami' 
when c.exclusions = '4' and c.pizza_id = 1 then 'Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami' 
when c.exclusions = '4' and c.pizza_id = 2 then 'Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce'
when c.extras = '1' and c.pizza_id = 1 then 'Bacon, Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami'
when c.extras = '1' and c.pizza_id = 2 then 'xBacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce' end) as ingredient
from customer_orders as c
join pizza_names as p on p.pizza_id = c.pizza_id
join runner_orders as r on r.order_id = c.order_id
where r.distance != 'null'
order by c.order_id asc)
select sum(round((length(Ingredient)- length(replace(Ingredient, "Bacon", "")))/length("Bacon"))) as Bacon_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "BBQ Sauce", "")))/length("BBQ Sauce"))) as BBQ_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Beef", "")))/length("Beef"))) as Beef_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Cheese", "")))/length("Cheese"))) as Cheese_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Chicken", "")))/length("Chicken"))) as Chicken_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Mushrooms", "")))/length("Mushrooms"))) as Mushrooms_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Onions", "")))/length("Onions"))) as Onions_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Pepperoni", "")))/length("Pepperoni"))) as Pepperoni_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Peppers", "")))/length("Peppers"))) as Peppers_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Salami", "")))/length("Salami"))) as Salami_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Tomatoes", "")))/length("Tomatoes"))) as Tomatoes_Frequency,
sum(round((length(Ingredient)- length(replace(Ingredient, "Tomato Sauce", "")))/length("Tomato Sauce"))) as TomatoSauce_Frequency
from cte
```
#### output
![Screen Shot 2022-09-05 at 3 19 18 AM](https://user-images.githubusercontent.com/92205707/188426790-71d76c4c-229e-4e00-a118-ff64747c16be.png)

### D. Pricing and Ratings

#### If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```mysql
with cte as (select c.pizza_id, p.pizza_name, count(c.pizza_id), 
(case when p.pizza_name = 'Meatlovers' then count(p.pizza_name) * 12
when p.pizza_name = 'Vegetarian' then count(p.pizza_name) * 10 end) as profits
from customer_orders as c
join pizza_names as p on p.pizza_id = c.pizza_id
join runner_orders as r on r.order_id = c.order_id
where r.distance != 'null'
group by c.pizza_id, p.pizza_name) 
select sum(profits) from cte as total_profits
```
#### output
![Screen Shot 2022-09-05 at 3 20 06 AM](https://user-images.githubusercontent.com/92205707/188426936-0ec71b84-cc65-45a4-8a97-b0149ef08ca7.png)

#### What if there was an additional $1 charge for any pizza extras?
#### 1.Add cheese is $1 extra
```mysql
set @base = 138;
select (LENGTH(group_concat(c.extras)) - LENGTH(REPLACE(group_concat(c.extras), ',', '')) + 1) + @base as Total
from customer_orders as c
join runner_orders as r
on c.order_id = r.order_id
where c.extras != 'null' and r.distance != 'null'; 
```
#### output
![Screen Shot 2022-09-05 at 3 21 04 AM](https://user-images.githubusercontent.com/92205707/188427117-c9e6efc1-28f6-4f43-9182-ee6b8565763f.png)

#### The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```mysql
select distinct r.order_id,
(case when r.distance/(r.duration/60) > 50 then '5'
when r.distance/(r.duration/60) > 40 then '4'
when r.distance/(r.duration/60) > 30 then '3'
when r.distance/(r.duration/60) > 20 then '2'
when r.distance/(r.duration/60) > 10 or r.distance = 'null' then '1' end) as 'rating (1 is lowest, 5 is highest)'
from customer_orders as c
join runner_orders as r on r.order_id = c.order_id
```
#### output
![Screen Shot 2022-09-05 at 3 21 47 AM](https://user-images.githubusercontent.com/92205707/188427249-7bee8978-f282-4e16-85cb-26218cad8bd0.png)

#### Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
-- 1.customer_id
-- 2.order_id
-- 3.runner_id
-- 4.rating
-- 5.order_time
-- 6.pickup_time
-- 7.Time between order and pickup
-- 8.Delivery duration
-- 9.Average speed
-- 10.Total number of pizzas
```mysql
with cte as (select c.customer_id, r.order_id, r.runner_id,
(case when r.distance/(r.duration/60) > 50 then '5'
when r.distance/(r.duration/60) > 40 then '4'
when r.distance/(r.duration/60) > 30 then '3'
when r.distance/(r.duration/60) > 20 then '2'
when r.distance/(r.duration/60) > 10 or r.distance = 'null' then '1' end) as rating,
c.order_time, r.pickup_time, TIMESTAMPDIFF(minute, c.order_time, r.pickup_time) as time_between_order_and_pickup, 
r.duration, r.distance/(r.duration/60) as avg_speed
from customer_orders as c
join runner_orders as r on r.order_id = c.order_id
where r.duration != 'null')
select customer_id, order_id, runner_id, rating, order_time, pickup_time, 
time_between_order_and_pickup, duration, round(avg_speed, 1), count(order_id) as 'Total number of pizzas' from cte
group by customer_id, order_id, runner_id, rating, order_time, pickup_time, time_between_order_and_pickup, duration, round(avg_speed, 1)
```
#### output
![Screen Shot 2022-09-05 at 3 22 39 AM](https://user-images.githubusercontent.com/92205707/188427426-3dea9203-8502-47ec-9792-741c5435b742.png)

#### If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```mysql
with revenue as (with cte1 as (with cte as (select c.order_id, c.pizza_id, p.pizza_name, count(c.pizza_id), 
(case when p.pizza_name = 'Meatlovers' then count(p.pizza_name) * 12
when p.pizza_name = 'Vegetarian' then count(p.pizza_name) * 10 end) as profits
from customer_orders as c
join pizza_names as p on p.pizza_id = c.pizza_id
join runner_orders as r on r.order_id = c.order_id
where r.distance != 'null'
group by c.order_id, c.pizza_id, p.pizza_name)
select cte.order_id, cte.pizza_id, cte.profits, r.distance from cte 
join runner_orders as r on r.order_id = cte.order_id)
select cte1.order_id, sum(profits) - sum(distinct distance) * 0.30 as revenue from cte1 group by cte1.order_id)
select round(sum(revenue),2) from revenue
```
#### output
![Screen Shot 2022-09-05 at 3 23 37 AM](https://user-images.githubusercontent.com/92205707/188427610-a0d90db6-686e-4f8f-84e6-58dec70c6cfb.png)

