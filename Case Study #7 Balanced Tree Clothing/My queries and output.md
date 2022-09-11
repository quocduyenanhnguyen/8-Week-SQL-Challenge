```mysql
use balanced_tree;
```
### High Level Sales Analysis

#### What was the total quantity sold for all products?
```mysql
with cte as (select s.prod_id, sum(s.qty) as total_quantity_sold from sales as s
join product_details as p on p.product_id = s.prod_id
group by s.prod_id)
select sum(total_quantity_sold) as total_sold from cte
```
#### output
![Screen Shot 2022-09-11 at 2 08 00 AM](https://user-images.githubusercontent.com/92205707/189519903-5462bf91-89ac-4a87-a14a-3efcca9020bc.png)

#### What is the total generated revenue for all products before discounts?
```mysql
with cte as (select s.prod_id, sum(s.qty * s.price) as revenue from sales as s
join product_details as p on p.product_id = s.prod_id
group by s.prod_id)
select sum(revenue) as total_revenue from cte
```
#### output
![Screen Shot 2022-09-11 at 2 08 38 AM](https://user-images.githubusercontent.com/92205707/189519919-955c8277-4b9f-4b26-ae8d-002c604909ff.png)

#### What was the total discount amount for all products?
```mysql
with cte1 as (with cte as (select s.prod_id, s.qty * s.price as revenue, s.discount from sales as s
join product_details as p on p.product_id = s.prod_id)
select prod_id, discount, round((revenue - revenue*(discount/100)),1) as amount_paid_after_discount_applied from cte)
select sum(amount_paid_after_discount_applied) as revenue_after_discount, sum(discount) as total_discount from cte1
```
#### output
![Screen Shot 2022-09-11 at 2 09 08 AM](https://user-images.githubusercontent.com/92205707/189519938-3d896217-0689-41e7-a2ea-e15359b98a18.png)

### Transaction Analysis

#### How many unique transactions were there?
```mysql
select count(distinct txn_id) from sales1
```
#### output
![Screen Shot 2022-09-11 at 2 09 52 AM](https://user-images.githubusercontent.com/92205707/189519964-4265f8d8-f6d4-4a9d-be07-2098ad17bf92.png)

#### What is the average unique products purchased in each transaction?
```mysql
with cte as (select txn_id, round(avg(qty), 1) as average from sales1
group by txn_id)
select avg(average) from cte
```
#### output
![Screen Shot 2022-09-11 at 2 17 42 AM](https://user-images.githubusercontent.com/92205707/189520248-a8f8d8ac-9f41-47bd-866c-7831b41a351a.png)

#### What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```mysql
create view percentile as select s.txn_id, round(sum(s.qty * s.price - s.qty * s.price * (s.discount/100)), 1) as revenue from sales1 as s
join product_details as p on p.product_id = s.prod_id
group by s.txn_id
```
#### 25th, 50th, and 75th percentile are done in Tableau. Output: 
![Screen Shot 2022-09-11 at 2 19 05 AM](https://user-images.githubusercontent.com/92205707/189520296-32bba960-5599-455a-b729-aea5d32855c1.png)
![Screen Shot 2022-09-11 at 2 19 15 AM](https://user-images.githubusercontent.com/92205707/189520300-2e857d6e-7b39-4096-ad6b-806b0f330207.png)
![Screen Shot 2022-09-11 at 2 19 25 AM](https://user-images.githubusercontent.com/92205707/189520305-294db108-f0ef-4839-bdac-9316b10da72d.png)

#### What is the average discount value per transaction?
```mysql
with cte as (select txn_id, round(avg(discount), 1) as average from sales1 group by txn_id)
select round(avg(average),1) as overall_average from cte
```
#### output
![Screen Shot 2022-09-11 at 2 20 07 AM](https://user-images.githubusercontent.com/92205707/189520340-0eb213e4-abe7-484f-9408-29a9b0f874c4.png)

#### What is the percentage split of all transactions for members vs non-members?
```mysql
with cte1 as (with cte as (select txn_id, sum(distinct member) as member from sales1 group by txn_id)
select sum(case when member = 1 then 1 else 0 end) as members_count,
sum(case when member = 0 then 1 else 0 end) as non_members_count
from cte)
select (members_count * 100)/(select count(distinct txn_id) from sales1) as members_count_percentage, 
(non_members_count * 100)/(select count(distinct txn_id) from sales1) as non_members_count_percentage
from cte1
```
#### output
![Screen Shot 2022-09-11 at 2 20 44 AM](https://user-images.githubusercontent.com/92205707/189520384-1e4b6a56-e62e-446b-9e3a-b152098092df.png)

#### What is the average revenue for member transactions and non-member transactions?
```mysql
with cte as (select txn_id, member, round(sum(qty * price - qty * price * (discount/100)), 1) as revenue from sales1 group by txn_id, member)
select member, (case when member = 1 then avg(revenue)
when member = 0 then avg(revenue) end) as non_members_and_members_revenue
from cte
group by member
```
#### output
![Screen Shot 2022-09-11 at 2 21 18 AM](https://user-images.githubusercontent.com/92205707/189520405-b06132b9-8359-4b89-946a-aa0ebf15e554.png)

### Product Analysis

#### What are the top 3 products by total revenue before discount?
```mysql
select p.product_name, sum(s.qty * s.price) as revenue from sales as s
join product_details as p on p.product_id = s.prod_id
group by p.product_name
order by revenue desc
limit 3
```
#### output
![Screen Shot 2022-09-11 at 2 21 54 AM](https://user-images.githubusercontent.com/92205707/189520426-43807f13-65ef-4132-827e-23419efcd5f4.png)

#### What is the total quantity, revenue and discount for each segment?
```mysql
select p.segment_name, sum(s.qty) as total_quantity, sum(s.qty * s.price) as revenue, sum(s.discount) as total_discount from sales as s
join product_details as p on p.product_id = s.prod_id
group by p.segment_name
```

#### output
![Screen Shot 2022-09-11 at 2 22 26 AM](https://user-images.githubusercontent.com/92205707/189520441-667003e5-64cd-4bb9-9132-c5978bf2ee9c.png)

#### What is the top selling product for each segment?
```mysql
with cte2 as (with cte1 as (with cte as (select p.segment_name, p.product_name, sum(s.qty) as quantity from sales as s
join product_details as p on p.product_id = s.prod_id
group by p.segment_name, p.product_name
order by quantity desc)
select segment_name, product_name, quantity, max(quantity) over (partition by segment_name) as top_total_quantity from cte)
select segment_name, product_name,
(case when quantity = top_total_quantity then quantity end) as top_selling_product from cte1)
select * from cte2 where top_selling_product is not null
```
#### output
![Screen Shot 2022-09-11 at 2 23 06 AM](https://user-images.githubusercontent.com/92205707/189520462-d2d72952-4699-4825-9268-1d3d190fdf22.png)

#### What is the total quantity, revenue and discount for each category?
```mysql
select p.category_id, sum(s.qty) as quantity, sum(s.price * s.qty) as revenue, sum(s.discount) as discount from sales as s
join product_details as p on p.product_id = s.prod_id
group by p.category_id
```
#### output
![Screen Shot 2022-09-11 at 2 23 38 AM](https://user-images.githubusercontent.com/92205707/189520491-f273b2c7-c7af-479d-b51d-83556bea7a03.png)

#### What is the top selling product for each category?
```mysql
with cte2 as (with cte1 as (with cte as (select p.category_name, p.product_name, sum(s.qty) as quantity from sales as s
join product_details as p on p.product_id = s.prod_id
group by p.category_name, p.product_name
order by quantity desc)
select category_name, product_name, quantity, max(quantity) over (partition by category_name) as top_total_quantity from cte)
select category_name, product_name,
(case when quantity = top_total_quantity then quantity end) as top_selling_product from cte1)
select * from cte2 where top_selling_product is not null
```
#### output
![Screen Shot 2022-09-11 at 2 24 09 AM](https://user-images.githubusercontent.com/92205707/189520519-58879f82-29ee-4905-8e38-dbe70d76c4e6.png)

#### What is the percentage split of revenue by product for each segment?
```mysql
with cte1 as (with cte as (select year(start_txn_time) as year, p.segment_name, p.product_name, round(sum(s.qty * s.price - s.qty * s.price * (s.discount/100)), 1) as revenue from sales1 as s
join product_details as p on p.product_id = s.prod_id
group by p.segment_name, p.product_name, year
order by revenue desc)
select year, segment_name, product_name, revenue, sum(revenue) over (partition by year) as total_revenue from cte)
select segment_name, product_name, round((revenue * 100)/total_revenue, 1) as percentage_split_revenue from cte1
```
#### output
![Screen Shot 2022-09-11 at 2 24 50 AM](https://user-images.githubusercontent.com/92205707/189520542-a8127d37-bf91-4699-aebd-68f1ec7e1108.png)

#### What is the percentage split of revenue by segment for each category?
```mysql
with cte1 as (with cte as (select year(start_txn_time) as year, p.category_name, p.segment_name, round(sum(s.qty * s.price - s.qty * s.price * (s.discount/100)), 1) as revenue from sales1 as s
join product_details as p on p.product_id = s.prod_id
group by p.category_name, p.segment_name, year
order by revenue desc)
select year, category_name, segment_name, revenue, sum(revenue) over (partition by year) as total_revenue from cte)
select category_name, segment_name, round((revenue * 100)/total_revenue, 1) as percentage_split_revenue from cte1
```
#### output
![Screen Shot 2022-09-11 at 2 25 29 AM](https://user-images.githubusercontent.com/92205707/189520560-514d3790-32d9-4ead-885a-57214b66cbda.png)

#### What is the percentage split of total revenue by category?
```mysql
with cte1 as (with cte as (select year(start_txn_time) as year, p.category_name, round(sum((s.qty * s.price) - (s.qty * s.price) * (s.discount/100)), 1) as revenue from sales1 as s
join product_details as p on p.product_id = s.prod_id
group by p.category_name, year
order by revenue desc)
select year, category_name, revenue, sum(revenue) over (partition by year) as total_revenue from cte)
select category_name, round((revenue * 100)/total_revenue, 1) as percentage_split_revenue from cte1
```
#### output
![Screen Shot 2022-09-11 at 2 25 59 AM](https://user-images.githubusercontent.com/92205707/189520575-b3c9a879-03c7-439b-87e0-addbec7b0942.png)

#### What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```mysql
select p.product_name, round(((select count(distinct s.txn_id)) * 100)/(select count(distinct txn_id) from sales1), 3) as penetration_rate 
from sales1 as s 
join product_details as p on s.prod_id = p.product_id
group by p.product_name
```
#### output
![Screen Shot 2022-09-11 at 2 26 34 AM](https://user-images.githubusercontent.com/92205707/189520596-3e32beaa-44ec-4dec-8608-e3eb4bf9f51b.png)

#### What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```mysql
with cte as (select s.txn_id, p.product_name from sales1 as s
join product_details as p on p.product_id = s.prod_id)
SELECT A.product_name, B.product_name , C.product_name, COUNT(1) as count
  FROM cte A
  JOIN cte B ON A.txn_id = B.txn_id AND A.product_name < B.product_name
  JOIN cte C ON A.txn_id = C.txn_id AND B.product_name < C.product_name
GROUP BY A.product_name, B.product_name , C.product_name
ORDER BY 4 DESC
```
#### output
![Screen Shot 2022-09-11 at 2 27 29 AM](https://user-images.githubusercontent.com/92205707/189520631-4924ee28-71c9-483a-a788-48e8da9e53f1.png)

### Bonus Challenge
#### Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.
#### Hint: you may want to consider using a recursive CTE to solve this problem!
```mysql
create view prices as select pr.product_id, pr.price, pr.product_name, pr.category_id, pr.segment_id, pr.style_id, 
pr.category_name, pr.segment_name, pr.style_name, 
ph.id, ph.parent_id, ph.level_text, ph.level_name
from product_details as pr
join product_prices as p on p.product_id = pr.product_id
right join product_hierarchy as ph on ph.id = p.id

with recursive report_structure(product_name, category_name, segment_name, style_name, id, parent_id, level_text, level_name) as
 (select product_name, category_name, segment_name, style_name, id, parent_id, level_text, level_name
  from prices where id = 15
 union all
   select pr.product_name, pr.category_name, pr.segment_name, pr.style_name, pr.id, pr.parent_id, pr.level_text, pr.level_name
    from prices as pr
    join report_structure rs on rs.parent_id = pr.id) select * from report_structure
 ```
 
#### output
![Screen Shot 2022-09-11 at 2 28 38 AM](https://user-images.githubusercontent.com/92205707/189520661-20e5cd36-9500-429e-ac3a-5b912e41e22a.png)
