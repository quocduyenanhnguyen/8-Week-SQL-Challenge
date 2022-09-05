```mysql
use foodie_fi
```
### A. Customer Journey

-- Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
-- Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
```mysql
with cte as (select plans.plan_id, plans.plan_name, plans.price, s.customer_id, s.start_date from plans 
join subscriptions as s on plans.plan_id = s.plan_id
where customer_id = 1 or customer_id = 2 or customer_id = 3 or customer_id = 4 or customer_id = 5 or customer_id = 6 or customer_id = 7 or
customer_id = 8)
select plan_id, plan_name, price, customer_id, start_date, datediff(start_date, 
lag(start_date) over (partition by customer_id order by plan_id)) as 'times it take to switch the plan' from cte
```
#### output
![Screen Shot 2022-09-05 at 3 31 46 AM](https://user-images.githubusercontent.com/92205707/188429024-f72b6e99-0da4-46fd-ac94-a5c7f89c1dd8.png)

-- most customers upgrade their plan to basic monthly within 7 days of free trial, two customers upgrade their plans to pro monthly plan
-- within 46 days and 100 days, respectively. Meanwhile, customer 4 and customer 6 stop using the plan after using basic monthly plan 
-- for 88 days and 58 days, respectively.

### B. Data Analysis Questions

#### How many customers has Foodie-Fi ever had?
```mysql
select count(distinct customer_id) from subscriptions
```
#### output
![Screen Shot 2022-09-05 at 3 32 56 AM](https://user-images.githubusercontent.com/92205707/188429240-47fc5aac-a464-4987-9089-1bcb7764b9dc.png)

#### What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```mysql
select count(plan_id) as trial_count, month(start_date) as month, LAST_DAY(start_date - interval 1 month) + interval 1 day as StartOfMonth
from subscriptions where plan_id = 0 
group by month(start_date), LAST_DAY(start_date - interval 1 month) + interval 1 day order by month(start_date) asc
```
#### output
![Screen Shot 2022-09-05 at 3 33 31 AM](https://user-images.githubusercontent.com/92205707/188429326-9f542ca3-f448-4103-b52e-a386f792b34e.png)

#### What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```mysql
select p.plan_name, count(p.plan_name) as count, year(s.start_date) as year from subscriptions as s
join plans as p on p.plan_id = s.plan_id
where year(s.start_date) != 2020
group by p.plan_name, year(s.start_date) 
order by count(p.plan_name) desc
```
#### output
![Screen Shot 2022-09-05 at 3 34 13 AM](https://user-images.githubusercontent.com/92205707/188429439-bccc8d72-3393-4e83-8f8b-681f03c40a92.png)

#### What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```mysql
select count(distinct customer_id) as count, 
round(100 * count(distinct customer_id)/(select count(distinct customer_id) from subscriptions),1) as churn_percent
from subscriptions where plan_id = 4
```
#### output
![Screen Shot 2022-09-05 at 3 34 45 AM](https://user-images.githubusercontent.com/92205707/188429554-5b1fda7b-91f6-4a14-8a4f-1f75cfac57a2.png)

#### How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```mysql
with count as (with cte as (select customer_id, plan_id, lead(plan_id) over (partition by customer_id order by plan_id) as lead_plan from subscriptions)
select customer_id, plan_id, lead_plan from cte where plan_id = 0 and lead_plan = 4)
select count(distinct customer_id) as those_that_churn_after_free_trial, 
round(100 * count(distinct customer_id)/(select count(distinct customer_id) from subscriptions),0)
from count
```
#### output
![Screen Shot 2022-09-05 at 3 35 17 AM](https://user-images.githubusercontent.com/92205707/188429642-f3793ad3-5586-48fd-b27e-dfc1b952d7b7.png)

#### What is the number and percentage of customer plans after their initial free trial?
```mysql
with cte as (select customer_id, plan_id, lead(plan_id) over (partition by customer_id order by plan_id) as lead_plan from subscriptions)
select lead_plan, count(plan_id) as count, round(100 * count(plan_id)/(select count(distinct customer_id) from subscriptions),1) as percentage_count 
from cte where plan_id = 0 and lead_plan = 1 or plan_id = 0 and lead_plan = 2 or plan_id = 0 and lead_plan = 3 or plan_id = 0 and lead_plan = 4
group by lead_plan
```
#### output
![Screen Shot 2022-09-05 at 3 35 59 AM](https://user-images.githubusercontent.com/92205707/188429745-9d7ce5c6-31ef-4076-935b-785fedcc72c9.png)

#### What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```mysql
select plan_id, count(distinct customer_id) as count, round(100 * count(distinct customer_id)/(select count(plan_id) 
from subscriptions where start_date <= '2020-12-31'),1) as percent_count from subscriptions where start_date <= '2020-12-31'
group by plan_id 
```
#### output
![Screen Shot 2022-09-05 at 3 36 55 AM](https://user-images.githubusercontent.com/92205707/188429901-8f3f11d3-8260-4d6c-98d5-8064160e0742.png)

#### How many customers have upgraded to an annual plan in 2020?
```mysql
with cte as (select *, year(start_date) as year from subscriptions)
select count(distinct customer_id) from cte where plan_id = 3 and year = 2020
```
#### output
![Screen Shot 2022-09-05 at 3 37 43 AM](https://user-images.githubusercontent.com/92205707/188430032-99ce58bb-b24c-45d4-88c5-89e54a1e6312.png)

#### How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```mysql
with average as (with cte2 as (with cte1 as (with cte as (select customer_id, plan_id, start_date, lead(plan_id) over (partition by customer_id order by plan_id) 
as plan_switch from subscriptions)
select customer_id, plan_id, start_date, plan_switch, 
datediff(lead(start_date) over (partition by customer_id order by start_date), start_date) as days_it_take_to_switch_to_annual_plan
from cte)
select customer_id, days_it_take_to_switch_to_annual_plan,
last_value(plan_id) over (partition by customer_id) as annual_plan
from cte1)
select customer_id, sum(days_it_take_to_switch_to_annual_plan) as total_days_take_to_switch_to_annual_plan 
from cte2 where annual_plan = 3 group by customer_id)
select avg(total_days_take_to_switch_to_annual_plan) from average
```
#### output
![Screen Shot 2022-09-05 at 3 38 21 AM](https://user-images.githubusercontent.com/92205707/188430157-fb20dc8c-0bfb-4dc1-862c-b6f27af3e200.png)

#### Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
-- average by month and year
```mysql
with cte6 as (with cte5 as (with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select customer_id, plan_id, start_date
from subscriptions) 
select customer_id, plan_id, start_date,
datediff(start_date, lag(start_date) over (partition by customer_id order by start_date)) as days_it_take_to_switch_to_annual_plan,
last_value(plan_id) over (partition by customer_id) as plan_switch
from cte where plan_id != 4)
select customer_id, start_date, days_it_take_to_switch_to_annual_plan, plan_id, plan_switch
from cte1)
select customer_id, start_date, sum(days_it_take_to_switch_to_annual_plan) as total, plan_switch 
from cte2 where plan_id = 3 or plan_switch = 3 group by customer_id, start_date, plan_switch)
select extract(year_month from start_date) as month_year, customer_id, sum(total) as total1, plan_switch 
from cte3 group by extract(year_month from start_date), customer_id, plan_switch)
select last_value(month_year) over (partition by customer_id) as month_year, customer_id, total1 from cte4)
select month_year, customer_id, sum(total1) as total2 from cte5 group by month_year, customer_id)
select month_year, count(customer_id) as count, round(avg(total2),1) as average from cte6 group by month_year
order by month_year asc
```
#### output
![Screen Shot 2022-09-05 at 3 39 01 AM](https://user-images.githubusercontent.com/92205707/188430294-033486f7-359d-4b77-ac95-a84013afa0f6.png)

#### average by month 
```mysql
with cte6 as (with cte5 as (with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select customer_id, plan_id, start_date
from subscriptions) 
select customer_id, plan_id, start_date,
datediff(start_date, lag(start_date) over (partition by customer_id order by start_date)) as days_it_take_to_switch_to_annual_plan,
last_value(plan_id) over (partition by customer_id) as plan_switch
from cte where plan_id != 4)
select customer_id, start_date, days_it_take_to_switch_to_annual_plan, plan_id, plan_switch
from cte1)
select customer_id, start_date, sum(days_it_take_to_switch_to_annual_plan) as total, plan_switch 
from cte2 where plan_id = 3 or plan_switch = 3 group by customer_id, start_date, plan_switch)
select month(start_date) as month_year, customer_id, sum(total) as total1, plan_switch 
from cte3 group by month(start_date), customer_id, plan_switch)
select last_value(month_year) over (partition by customer_id) as month_year, customer_id, total1 from cte4)
select month_year, customer_id, sum(total1) as total2 from cte5 group by month_year, customer_id)
select month_year, count(customer_id) as count, round(avg(total2),1) as average from cte6 group by month_year
order by month_year asc
```
#### output
![Screen Shot 2022-09-05 at 3 39 49 AM](https://user-images.githubusercontent.com/92205707/188430431-6762a4c8-355e-4a26-96a9-502962430fab.png)

#### How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```mysql
with cte2 as (with cte1 as (with cte as (select customer_id, plan_id, start_date, year(start_date) as year from subscriptions where year(start_date) = 2020)
select customer_id, plan_id, start_date, last_value(plan_id) over (partition by customer_id) as last_value1 from cte)
select * from cte1 where plan_id = 2 and last_value1 = 1) select count(*) as 'number of customers downgraded from 2 to 1' from cte2
```
#### output
![Screen Shot 2022-09-05 at 3 40 26 AM](https://user-images.githubusercontent.com/92205707/188430547-78f4f23d-d9ff-4463-ab74-5f3aa8a8ece7.png)

### D. Outside The Box Questions

-- The following are open ended questions which might be asked during a technical interview for this case study - there are no right or wrong answers, but answers that make sense from both a technical and a business perspective make an amazing impression!

-- How would you calculate the rate of growth for Foodie-Fi?
```mysql
create view plan as with cte3 as (with cte2 as (with cte1 as (with cte as (select s.customer_id, s.plan_id, s.start_date, year(s.start_date) as year, p.price from subscriptions as s
join plans as p on p.plan_id = s.plan_id where s.plan_id != 0)
select *, last_value(plan_id) over (partition by customer_id) as latest_plan, lead(start_date) over (partition by customer_id) 
as latest_date, last_value(start_date) over (partition by customer_id) as latest_date1,
year(last_value(start_date) over (partition by customer_id)) as latest_year1
from cte)
select *, TIMESTAMPDIFF(month, start_date, latest_date) as number_of_months_enrolled_before_switch_plan from cte1)
select *, ((number_of_months_enrolled_before_switch_plan + 1) * price) as payment_made_before_switch_plan from cte2)
select customer_id, plan_id, start_date, year, price, latest_plan, latest_date, latest_date1, latest_year1,
number_of_months_enrolled_before_switch_plan, payment_made_before_switch_plan,
(case when plan_id = 1 and latest_plan != 4 and latest_year1 = 2020 or payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1 + 12) 
when plan_id = 2 and latest_plan != 4 and latest_year1 = 2020 or payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1 + 12)
when plan_id = 3 and latest_plan != 4 and latest_year1 = 2020 or payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1 + 12)
when plan_id = 1 and latest_plan = 4 then price * (number_of_months_enrolled_before_switch_plan + 1)
when plan_id = 2 and latest_plan = 4 then price * (number_of_months_enrolled_before_switch_plan + 1)
when plan_id = 3 and latest_plan = 4 then price * (number_of_months_enrolled_before_switch_plan + 1)
when plan_id = 1 and latest_plan != 4 or latest_year1 = 2021 and payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1)
when plan_id = 2 and latest_plan != 4 or latest_year1 = 2021 and payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1)
when plan_id = 3 and latest_plan != 4 or latest_year1 = 2021 and payment_made_before_switch_plan = NULL then price * (month(last_day('2021-12-31')) - month(latest_date1) + 1)
end) as total_payment_as_ending_value_until_end_of_2021 from cte3
```
-- growth rate calculation
```mysql
with cte as (select customer_id, plan_id, price, start_date, first_value(price) over (partition by customer_id) as beginning_value,
(case when payment_made_before_switch_plan then price * (number_of_months_enrolled_before_switch_plan + 1)
else total_payment_as_ending_value_until_end_of_2021 end) as total_payment_in_2021
from plan)
select customer_id, round(100 * (sum(total_payment_in_2021) - beginning_value)/beginning_value,0) as growth_rate
from cte group by customer_id, beginning_value
```
#### output
![Screen Shot 2022-09-05 at 3 42 47 AM](https://user-images.githubusercontent.com/92205707/188430943-0f9971b8-f862-4d46-b50c-ecfc03998256.png)

```mysql
with cte as (select customer_id, plan_id, price, start_date, first_value(price) over (partition by customer_id, plan_id) as beginning_value,
(case when payment_made_before_switch_plan then price * (number_of_months_enrolled_before_switch_plan + 1)
else total_payment_as_ending_value_until_end_of_2021 end) as total_payment_in_2021
from plan)
select customer_id, plan_id, round(100 * (total_payment_in_2021 - beginning_value)/beginning_value,0) as growth_rate
from cte group by customer_id, plan_id, beginning_value, total_payment_in_2021
```
#### output
![Screen Shot 2022-09-05 at 3 43 28 AM](https://user-images.githubusercontent.com/92205707/188431049-b13aefbd-44ce-4a60-9916-36815bf3087a.png)

```mysql
with cte1 as (with cte as (select customer_id, plan_id, price, start_date, first_value(price) over (partition by customer_id, plan_id) as beginning_value,
(case when payment_made_before_switch_plan then price * (number_of_months_enrolled_before_switch_plan + 1)
else total_payment_as_ending_value_until_end_of_2021 end) as total_payment_in_2021
from plan)
select plan_id, beginning_value, sum(total_payment_in_2021) as ending_value
from cte group by plan_id, beginning_value)
select plan_id, round(100 * (ending_value - beginning_value)/beginning_value,0) as growth_rate
from cte1 
```
#### output
![Screen Shot 2022-09-05 at 3 43 51 AM](https://user-images.githubusercontent.com/92205707/188431110-71e06afc-3ed5-4aab-a06c-874ebdbbe464.png)

-- I would calculate the growth rate by each customer to see which customer brings the most profits till the end of 2021 and I would also calculate the growth rate by each plan to see which plan sees the fastest growth 
