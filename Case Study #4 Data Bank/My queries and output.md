```mysql
use data_bank
```

### A. Customer Nodes Exploration

#### How many unique nodes are there on the Data Bank system?
```mysql
select count(distinct node_id) from customer_nodes
```
#### output
![Screen Shot 2022-09-05 at 3 52 09 AM](https://user-images.githubusercontent.com/92205707/188432528-1abc310a-3e7a-4977-97ca-af0cb180ff35.png)

#### What is the number of nodes per region?
```mysql
select region_id, count(node_id) from customer_nodes group by region_id
```
#### output
![Screen Shot 2022-09-05 at 3 52 45 AM](https://user-images.githubusercontent.com/92205707/188432636-130aef23-0e9d-4c46-9460-ce1c75703550.png)

#### How many customers are allocated to each region?
```mysql
select region_id, count(distinct customer_id) from customer_nodes group by region_id
```
#### output
![Screen Shot 2022-09-05 at 3 53 20 AM](https://user-images.githubusercontent.com/92205707/188432737-51163d67-284b-4281-81c0-5395ff129ac6.png)

#### How many days on average are customers reallocated to a different node?
```mysql
with cte1 as (with cte as (select *, TIMESTAMPDIFF(day, start_date, end_date) as days_between
from customer_nodes where end_date != '9999-12-31' order by customer_id asc)
select customer_id, region_id, node_id, sum(days_between) as days_between from cte
group by customer_id, region_id, node_id order by customer_id asc)
select round(avg(days_between), 1) from cte1
```
#### output
![Screen Shot 2022-09-05 at 3 53 49 AM](https://user-images.githubusercontent.com/92205707/188432829-a1a44ab6-4f15-433c-bdc6-15b2416551cc.png)

#### What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```mysql
create view cte2 as (with cte1 as (with cte as (select *, TIMESTAMPDIFF(day, start_date, end_date) as days_between
from customer_nodes where end_date != '9999-12-31' order by customer_id asc)
select customer_id, region_id, node_id, sum(days_between) as days_between from cte
group by customer_id, region_id, node_id order by customer_id asc)
select region_id, customer_id, round(avg(days_between), 1) as average from cte1 group by region_id, customer_id order by region_id asc)
```
#### output
-- median by each region
```mysql
SELECT region_id, AVG(average) as median
FROM (SELECT region_id, average, (SELECT count(*) FROM cte2 t2 WHERE t2.region_id = t3.region_id) as ct,
seq, (SELECT count(*) FROM cte2 t2 WHERE t2.region_id < t3.region_id) as delta
FROM (SELECT region_id, average, @rownum := @rownum + 1 as seq
FROM (SELECT * FROM cte2 ORDER BY region_id, average) t1 ORDER BY region_id, average) t3 
CROSS JOIN (SELECT @rownum := 0) x HAVING (ct%2 = 0 and seq-delta between floor((ct+1)/2) and floor((ct+1)/2) +1)
                  or (ct%2 <> 0 and seq-delta = (ct+1)/2)) T
GROUP BY region_id ORDER BY region_id;
```
#### output
![Screen Shot 2022-09-05 at 3 55 01 AM](https://user-images.githubusercontent.com/92205707/188433042-99919c9c-d658-4af5-938b-ddeb80860884.png)

-- percentile by each region (I solved this in Tableau)
#### output
![Screen Shot 2022-09-05 at 3 57 03 AM](https://user-images.githubusercontent.com/92205707/188433380-2f3ba155-0d52-49d9-b3af-805bc16e5b8e.png)
![Screen Shot 2022-09-05 at 3 57 13 AM](https://user-images.githubusercontent.com/92205707/188433414-988964c7-8a73-4684-b7c9-9526fbf92814.png)

### B. Customer Transactions

#### What is the unique count and total amount for each transaction type?
```mysql
select txn_type, count(distinct customer_id), sum(txn_amount) from customer_transactions group by txn_type
```
#### output
![Screen Shot 2022-09-05 at 3 58 02 AM](https://user-images.githubusercontent.com/92205707/188433560-c70126d3-317c-4dcf-80bd-512f611383c1.png)

#### What is the average total historical deposit counts and amounts for all customers?
-- total deposit amounts by each customer, then average deposit amounts for all customers
```mysql
with cte as (select customer_id, txn_type, count(txn_type) as deposit_counts, sum(txn_amount) as amount_deposit from customer_transactions 
where txn_type = 'deposit' group by customer_id, txn_type)
select round(avg(deposit_counts),0), avg(amount_deposit) from cte
```
#### output
![Screen Shot 2022-09-05 at 3 58 39 AM](https://user-images.githubusercontent.com/92205707/188433682-6c76bad0-40ee-48d2-a5ed-eec8e1e38551.png)

-- average deposit amounts for all customers
```mysql
with cte as (select customer_id, txn_type, count(txn_type) as deposit_counts, avg(txn_amount) as amount_deposit from customer_transactions 
where txn_type = 'deposit' group by customer_id, txn_type)
select round(avg(deposit_counts),0), avg(amount_deposit) from cte
```
#### output
![Screen Shot 2022-09-05 at 3 59 14 AM](https://user-images.githubusercontent.com/92205707/188433770-05cb2f02-cd05-4a02-b7d6-98daf52b2c24.png)

#### For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```mysql
with cte2 as (with cte1 as (with cte as (select month(txn_date) as month, customer_id, 
(case when txn_type = 'deposit' then count(txn_type) end) as deposit_count,
(case when txn_type = 'purchase' then count(txn_type) end) as purchase_count,
(case when txn_type = 'withdrawal' then count(txn_type) end) as withdrawal_count from customer_transactions
group by customer_id, txn_type, txn_date)
select month, customer_id, sum(deposit_count) as total_deposit_count, sum(purchase_count) as total_purchase_count, 
sum(withdrawal_count) as total_withdrawal_count
from cte group by month, customer_id)
select month, customer_id from cte1 where total_deposit_count > 1 and (total_purchase_count = 1 or total_withdrawal_count = 1))
select month, count(customer_id) from cte2 group by month order by month asc
```
#### output
![Screen Shot 2022-09-05 at 3 59 54 AM](https://user-images.githubusercontent.com/92205707/188433900-bad0a496-f242-4319-bec9-26349fe00d89.png)

#### What is the closing balance for each customer at the end of the month?
```mysql
with cte6 as (with cte5 as (with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select *, month(txn_date) as month, 
year(txn_date) as year2020 from customer_transactions)
select year2020, month, customer_id, txn_type, sum(txn_amount) as total_amount from cte group by year2020, month, customer_id, txn_type)
select year2020, month, customer_id, txn_type,
(case when txn_type = 'deposit' then sum(total_amount) end) as total_deposit,
(case when txn_type = 'purchase' then sum(total_amount) end) as total_purchase,
(case when txn_type = 'withdrawal' then sum(total_amount) end) as total_withdrawal from cte1
group by year2020, month, customer_id, txn_type)
select year2020, month, customer_id, sum(total_deposit) as total_deposit1, sum(total_purchase) as total_purchase1, 
sum(total_withdrawal) as total_withdrawal1 from cte2 group by month, customer_id, year2020)
select year2020, month, customer_id, total_deposit1, total_purchase1, total_withdrawal1,
(case when total_deposit1 then total_deposit1 else 0 end) as deposit,
(case when total_purchase1 then total_purchase1 else 0 end) as purchase,
(case when total_withdrawal1 then total_withdrawal1 else 0 end) as withdrawal
from cte3)
select year2020, month, customer_id, (deposit - purchase - withdrawal) as closing_balance from cte4)
select year2020, max(month) over (partition by customer_id) as latest_month, customer_id, closing_balance from cte5
group by customer_id, closing_balance, year2020, month)
select a.April as ending_balance_as_of_April, c.latest_month, c.customer_id, sum(c.closing_balance) as ending_balance from cte6 as c
cross join april as a on a.year2020 = c.year2020
group by c.latest_month, c.customer_id, a.April
order by c.customer_id desc
```
#### output
![Screen Shot 2022-09-05 at 4 00 48 AM](https://user-images.githubusercontent.com/92205707/188434040-164d95eb-79b3-465e-9681-6300ff20f530.png)

#### What is the percentage of customers who increase their closing balance by more than 5%?
-- I created view named balance based on cte4 from the previous question before transforming data
-- count number of percentage increase then calculate percentage of customers with closing balance_increase > 5
```mysql
with cte2 as (with cte1 as (with cte as (select month, customer_id, sum(closing_balance) over (partition by customer_id order by month asc) as closing_balance
from balance) select month, customer_id, closing_balance, lead(closing_balance) over (partition by customer_id order by month) as balance_next_month
from cte order by customer_id, month asc)
select month, customer_id, ((balance_next_month - closing_balance) * 100)/closing_balance as percentage_increase_next_month from cte1)
select round(100 * count(percentage_increase_next_month)/(select count(percentage_increase_next_month) from cte2),1) 
from cte2 where percentage_increase_next_month > 5
```
#### output
![Screen Shot 2022-09-05 at 4 01 26 AM](https://user-images.githubusercontent.com/92205707/188434135-caaa13da-6f5c-4dca-8cf6-7e25285a1fae.png)

-- count number of unique customers then calculate percentage of customers with closing balance_increase > 5%
```mysql
with cte2 as (with cte1 as (with cte as (select month, customer_id, sum(closing_balance) over (partition by customer_id order by month asc) as closing_balance
from balance) select month, customer_id, closing_balance, lead(closing_balance) over (partition by customer_id order by month) as balance_next_month
from cte order by customer_id, month asc)
select month, customer_id, ((balance_next_month - closing_balance) * 100)/closing_balance as percentage_increase_next_month from cte1)
select round(100 * count(distinct customer_id)/(select count(distinct customer_id) from cte2),1) 
from cte2 where percentage_increase_next_month > 5
```
#### output
![Screen Shot 2022-09-05 at 4 01 56 AM](https://user-images.githubusercontent.com/92205707/188434229-fc8cd123-c843-4cc8-bf01-b9affb898d19.png)

-- count number of unique customers then calculate percentage of customers with closing balance_increase > 5% next month, i.e. March
-- because some customers don't have any transaction activities in March so we will count their percentage_increase_next_month instead
```mysql
with cte2 as (with cte1 as (with cte as (select month, customer_id, sum(closing_balance) over (partition by customer_id order by month asc) as closing_balance
from balance) select month, customer_id, closing_balance, lead(closing_balance) over (partition by customer_id order by month) as balance_next_month
from cte order by customer_id, month asc)
select month, customer_id, ((balance_next_month - closing_balance) * 100)/closing_balance as percentage_increase_next_month from cte1)
select round(100 * count(percentage_increase_next_month)/(select count(percentage_increase_next_month) from cte2 where month = 2),1) 
from cte2 where percentage_increase_next_month > 5 and month = 2
```
#### output
![Screen Shot 2022-09-05 at 4 03 00 AM](https://user-images.githubusercontent.com/92205707/188434367-00d0bff8-7861-4b15-aa4e-b029749b68ca.png)


