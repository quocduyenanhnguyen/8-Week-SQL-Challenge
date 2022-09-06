```mysql
use clique_bait
```

### 2. Digital Analysis

#### How many users are there?
```mysql
select count(distinct user_id) from users
```
#### output
![Screen Shot 2022-09-05 at 7 56 02 PM](https://user-images.githubusercontent.com/92205707/188537432-1ff7b4aa-a092-4b6d-b2e3-2e87d65b1968.png)

#### How many cookies does each user have on average?
```mysql
with cte as (select distinct user_id, count(cookie_id) as count from users group by user_id)
select round(avg(count), 1) from cte
```
#### output
![Screen Shot 2022-09-05 at 7 56 32 PM](https://user-images.githubusercontent.com/92205707/188537500-fc96c132-9d68-4f07-8d52-4a9655e06c4a.png)

#### What is the unique number of visits by all users per month?
```mysql
select month(event_time) as month, count(distinct visit_id) as count from events group by month(event_time)
```
#### output
![Screen Shot 2022-09-05 at 7 57 05 PM](https://user-images.githubusercontent.com/92205707/188537555-706fa809-456b-4719-828c-260251d50b16.png)

#### What is the number of events for each event type?
```mysql
select e.event_name, count(events.event_type) from events 
join event_identifier as e on e.event_type = events.event_type
group by e.event_name
```
#### output
![Screen Shot 2022-09-05 at 7 57 52 PM](https://user-images.githubusercontent.com/92205707/188537645-504e7c16-c0d3-4016-ba56-87b8ddc8da86.png)

#### What is the percentage of visits which have a purchase event?
```mysql
with cte as (select e.event_name, count(distinct events.visit_id) as count from events 
join event_identifier as e on e.event_type = events.event_type
group by e.event_name)
select round((count * 100)/(select count(distinct visit_id) from events), 1) from cte where event_name = 'Purchase'
```
#### output
![Screen Shot 2022-09-05 at 7 58 25 PM](https://user-images.githubusercontent.com/92205707/188537730-43dee983-2585-454b-b290-4a78c1940301.png)

#### What is the percentage of visits which view the checkout page but do not have a purchase event?
```mysql
with cte as (select count(distinct visit_id) as count from events where visit_id not in (select visit_id from events where event_type = 3) 
and event_type = 1 and page_id = 12)
select round((100 * count)/(select count(distinct visit_id) from events), 1) from cte
```
#### output
![Screen Shot 2022-09-05 at 7 58 58 PM](https://user-images.githubusercontent.com/92205707/188537807-97af783a-2d9f-47a3-85a3-c6ac01b57af3.png)

#### What are the top 3 pages by number of views?
```mysql
select p.page_name, count(e.event_type) as count from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where e.event_type = 1
group by p.page_name
order by count desc
limit 3
```
#### output
![Screen Shot 2022-09-05 at 7 59 31 PM](https://user-images.githubusercontent.com/92205707/188537877-44c336e4-d3d7-47ea-bbb2-28fab39c6490.png)

#### What is the number of views and cart adds for each product category?
```mysql
select distinct p.product_category,
sum(case when e.event_type = 2 then 1 else 0 end) as count_cart_adds,
sum(case when e.event_type = 1 then 1 else 0 end) as count_views
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_category IS NOT NULL
group by p.product_category
```
#### output
![Screen Shot 2022-09-05 at 8 00 22 PM](https://user-images.githubusercontent.com/92205707/188537998-fbc7ff42-2df0-4906-afbe-03ffcc06e068.png)

#### What are the top 3 products by purchases?
```mysql
with cte as (select e.visit_id, p.product_category, p.page_name, p.product_id, e.page_id, e.event_type, ev.event_name
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type)
select product_id, page_name, count(event_name) from cte
where product_id is not null and visit_id in (select visit_id from events where event_type = 3) and event_name = 'Add to Cart'
group by product_id, page_name
order by count(event_name) desc
limit 3
```
#### output
![Screen Shot 2022-09-05 at 8 00 59 PM](https://user-images.githubusercontent.com/92205707/188538068-f3d083c8-dc99-4219-8c9c-d2d7957fca4a.png)

### 3. Product Funnel Analysis
-- Using a single SQL query - create a new output table which has the following details:

##### How many times was each product viewed?
##### How many times was each product added to cart?
##### How many times was each product added to a cart but not purchased (abandoned)?
##### How many times was each product purchased?
##### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products

-- count individual products
```mysql
select p.page_id, p.page_name, p.product_id, p.product_category,
sum(case when ev.event_name = 'Page View' then 1 else 0 end) as page_view_count,
sum(case when ev.event_name = 'Add to Cart' then 1 else 0 end) as cart_adds_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id not in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as cart_adds_no_purchase_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as purchase_count
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_id is not null
group by p.page_name, p.product_id, p.product_category, p.page_id
```
#### output
![Screen Shot 2022-09-05 at 8 03 12 PM](https://user-images.githubusercontent.com/92205707/188538330-42e2d18d-63e4-4937-863c-a3044f596a7b.png)

-- count product category
```mysql
select p.product_category,
sum(case when ev.event_name = 'Page View' then 1 else 0 end) as page_view_count,
sum(case when ev.event_name = 'Add to Cart' then 1 else 0 end) as cart_adds_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id not in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as cart_adds_no_purchase_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as purchase_count
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_id is not null
group by p.product_category
```
#### output
![Screen Shot 2022-09-05 at 8 03 54 PM](https://user-images.githubusercontent.com/92205707/188538397-a5b1c9ca-0acd-43a5-b528-4036eb360e5d.png)

-- Use your 2 new output tables - answer the following questions:

##### Which product had the most views, cart adds and purchases?
-- most views are oysters, most cart adds are lobsters, most purchases are lobsters

##### Which product was most likely to be abandoned?
-- Russian Caviar

##### Which product had the highest view to purchase percentage?
```mysql
with cte as (select p.page_id, p.page_name, p.product_id, p.product_category,
sum(case when ev.event_name = 'Page View' then 1 else 0 end) as page_view_count,
sum(case when ev.event_name = 'Add to Cart' then 1 else 0 end) as cart_adds_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id not in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as cart_adds_no_purchase_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as purchase_count
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_id is not null
group by p.page_name, p.product_id, p.product_category, p.page_id)
select page_name, round(100 * purchase_count/page_view_count, 1) as purchase_percentage from cte order by purchase_percentage desc
limit 1
```
#### output
![Screen Shot 2022-09-05 at 8 05 04 PM](https://user-images.githubusercontent.com/92205707/188538544-db977586-d436-46a3-a01d-d5d6b7e1d89c.png)

##### What is the average conversion rate from view to cart add?
```mysql
with cte1 as (with cte as (select p.page_id, p.page_name, p.product_id, p.product_category,
sum(case when ev.event_name = 'Page View' then 1 else 0 end) as page_view_count,
sum(case when ev.event_name = 'Add to Cart' then 1 else 0 end) as cart_adds_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id not in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as cart_adds_no_purchase_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as purchase_count
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_id is not null 
group by p.page_name, p.product_id, p.product_category, p.page_id)
select page_name, round(100 * cart_adds_count/page_view_count, 1) as view_to_add_conversion
from cte order by view_to_add_conversion desc)
select round(avg(view_to_add_conversion), 1) from cte1
```
#### output
![Screen Shot 2022-09-05 at 8 05 48 PM](https://user-images.githubusercontent.com/92205707/188538617-95bb8518-28af-4127-88ee-9f8f4f433ae5.png)

##### What is the average conversion rate from cart add to purchase?
```mysql
with cte1 as (with cte as (select p.page_id, p.page_name, p.product_id, p.product_category,
sum(case when ev.event_name = 'Page View' then 1 else 0 end) as page_view_count,
sum(case when ev.event_name = 'Add to Cart' then 1 else 0 end) as cart_adds_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id not in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as cart_adds_no_purchase_count,
sum(case when ev.event_name = 'Add to Cart' and e.visit_id in (select visit_id from events where event_type = 3) then 1 else 0 end) 
as purchase_count
from events as e
join page_hierarchy as p on p.page_id = e.page_id
join event_identifier as ev on ev.event_type = e.event_type
where p.product_id is not null 
group by p.page_name, p.product_id, p.product_category, p.page_id)
select page_name, round(100 * purchase_count/cart_adds_count, 1) as add_to_purchase_conversion
from cte order by add_to_purchase_conversion desc)
select round(avg(add_to_purchase_conversion), 1) from cte1
```
#### output
![Screen Shot 2022-09-05 at 8 06 24 PM](https://user-images.githubusercontent.com/92205707/188538677-1472ad68-f41d-45c1-b0c3-b70f1f193d49.png)

### 4. Campaigns Analysis
##### Generate a table that has 1 single row for every unique visit_id record and has the following columns:

##### user_id
##### visit_id
##### visit_start_time: the earliest event_time for each visit
##### page_views: count of page views for each visit
##### cart_adds: count of product cart add events for each visit
##### purchase: 1/0 flag if a purchase event exists for each visit
##### campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
##### impression: count of ad impressions for each visit
##### click: count of ad clicks for each visit
##### cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
```mysql
create view cart_products as select e.visit_id, 
group_concat(p.page_name order by e.sequence_number asc SEPARATOR ', ') as cart_products 
from events as e
cross join page_hierarchy as p on p.page_id = e.page_id
where e.event_type = 2
group by e.visit_id

with cte1 as (with cte as (select distinct e.visit_id, u.user_id, min(e.event_time) as visit_start_time, 
sum(case when e.event_type = 1 then 1 else 0 end) as page_views,
sum(case when e.event_type = 2 then 1 else 0 end) as cart_adds,
sum(case when e.event_type = 3 then 1 else 0 end) as purchase,
sum(case when e.event_type = 4 then 1 else 0 end) as impression,
sum(case when e.event_type = 5 then 1 else 0 end) as click
from events as e
join users as u on u.cookie_id = e.cookie_id
join event_identifier as ev on ev.event_type = e.event_type
group by e.visit_id, u.user_id)
select cte.visit_id, cte.user_id, cte.visit_start_time, cte.page_views, cte.cart_adds, cte.purchase, c.campaign_name, cte.impression,
cte.click
from cte 
left join campaign_identifier as c on cte.visit_start_time between c.start_date and c.end_date)
select cte1.visit_id, cte1.user_id, cte1.visit_start_time, cte1.page_views, cte1.cart_adds, cte1.purchase, cte1.campaign_name, cte1.impression,
cte1.click, c.cart_products 
from cte1
left join cart_products as c on c.visit_id = cte1.visit_id
```
#### output
![Screen Shot 2022-09-05 at 8 08 19 PM](https://user-images.githubusercontent.com/92205707/188538846-96104bd3-2283-4773-a2a6-10f875a7f827.png)




