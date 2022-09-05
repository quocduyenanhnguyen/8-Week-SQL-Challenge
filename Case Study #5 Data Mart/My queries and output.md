```mysql
use data_mart
```

### 2. Data Exploration

#### What day of the week is used for each week_date value?
```mysql
select week_date, dayofweek(week_date) from weekly_sales_cleaned
```
#### output (we use 2 which is Monday for all values)
![Screen Shot 2022-09-05 at 4 11 58 AM](https://user-images.githubusercontent.com/92205707/188435882-f019d43b-bc43-4c03-8b08-24f877ca184e.png)

#### What range of week numbers are missing from the dataset?
```mysql
select 52 - count(distinct week_num) from weekly_sales_cleaned 
```
#### output
![Screen Shot 2022-09-05 at 4 12 56 AM](https://user-images.githubusercontent.com/92205707/188436025-896e767b-a06e-4e93-b215-eeb09c6c407e.png)

#### How many total transactions were there for each year in the dataset?
```mysql
select year, sum(transactions) from weekly_sales_cleaned group by year
```
#### output
![Screen Shot 2022-09-05 at 4 13 26 AM](https://user-images.githubusercontent.com/92205707/188436105-a05ce043-f765-498b-ae0a-e81a6911b490.png)

#### What is the total sales for each region for each month?
```mysql
select month, region, sum(sales) from weekly_sales_cleaned group by month, region order by region, month asc
```
#### output
![Screen Shot 2022-09-05 at 4 14 04 AM](https://user-images.githubusercontent.com/92205707/188436221-5d029d16-e2ec-4713-a399-be307f053623.png)

#### What is the total count of transactions for each platform
```mysql
select platform, sum(transactions) from weekly_sales_cleaned group by platform
```
#### output
![Screen Shot 2022-09-05 at 4 14 43 AM](https://user-images.githubusercontent.com/92205707/188436343-5283799f-eb5d-4120-8909-ab5383ca5bca.png)

#### What is the percentage of sales for Retail vs Shopify for each month?
```mysql
with cte as (select month, platform, sum(sales) as total_sales from weekly_sales_cleaned group by month, platform order by month asc)
select month, platform, round(total_sales * 100/sum(total_sales) over (partition by month),3) as percentage_of_sales from cte 
```
#### output
![Screen Shot 2022-09-05 at 4 15 27 AM](https://user-images.githubusercontent.com/92205707/188436455-b9b07285-6a77-4707-8f56-8cc72f9f3d10.png)

#### What is the percentage of sales by demographic for each year in the dataset?
```mysql
with cte as (select year, demographic, sum(sales) as total_sales from weekly_sales_cleaned group by year, demographic)
select year, demographic, round(total_sales * 100/sum(total_sales) over (partition by year), 2) as percentage_of_sales from cte
```
#### output
![Screen Shot 2022-09-05 at 4 16 06 AM](https://user-images.githubusercontent.com/92205707/188436572-3685eeb8-76ca-4a9d-a790-74fe3f06aeb9.png)

#### Which age_band and demographic values contribute the most to Retail sales?
```mysql
with cte as (select age_band, demographic, platform, sum(sales) as total_sales from weekly_sales_cleaned where platform = 'Retail'
group by age_band, demographic, platform)
select age_band, demographic, total_sales, round(100 * total_sales/sum(total_sales) over (partition by platform), 2) 
as percentage_sales from cte order by total_sales desc
```
#### output
![Screen Shot 2022-09-05 at 4 16 44 AM](https://user-images.githubusercontent.com/92205707/188436672-6d32ccf3-c3eb-4eb3-8b62-c27baee81445.png)

#### Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```mysql
select year, platform, round(avg(avg_transaction),0) as average_method_1,
round(avg(sales)/avg(transactions),0) as average_method_2
from weekly_sales_cleaned group by year, platform
```
#### output
![Screen Shot 2022-09-05 at 4 17 19 AM](https://user-images.githubusercontent.com/92205707/188436789-f176212b-b813-4f8b-9afb-e5543e0518eb.png)

### 3. Before & After Analysis

-- This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time. Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect. We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before. Using this analysis approach - answer the following questions:

-- What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

-- 2020-06-15 is week 24

-- growth rate group by
```mysql
with cte2 as (with cte1 as (with cte as (select week_date, week_num, month, year, region, platform, segment, age_band, demographic, customer_type, transactions, sales, 
avg_transaction from weekly_sales_cleaned where year = 2020
and week_num between 20 and 27 order by week_date asc)
select year, week_num, sum(sales) as sales from cte group by year, week_num)
select week_num, sales as ending_value, lead(sales) over (partition by year) as beginning_value, 
round(100 * sales/sum(sales) over (partition by year), 1) as percentage_sales
from cte1)
select week_num, round(100 * (ending_value - beginning_value)/beginning_value,2) as growth_rate, percentage_sales from cte2
```
#### output
![Screen Shot 2022-09-05 at 4 18 12 AM](https://user-images.githubusercontent.com/92205707/188436897-92f53ece-a043-42cf-b38a-d49cb7f91aa4.png)

-- overall growth rate
```mysql
with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select week_date, week_num, month, year, region, platform, segment, age_band, demographic, customer_type, transactions, sales, 
avg_transaction from weekly_sales_cleaned where year = 2020
and week_num between 20 and 27 order by week_date asc)
select year, week_num, sum(sales) as sales from cte group by year, week_num)
select week_num, year, (case when week_num < 24 then 'Before' else 'After' end) as status, sales from cte1)
select status, year, sum(sales) as total from cte2 group by status, year)
select status, round(100 * total/sum(total) over (partition by year), 1) as percentage_sales, total,
lead(total) over (partition by year) as total1
from cte3)
select status, percentage_sales, total, round(100 * (total - total1)/(total1), 2) as growth_rate from cte4
```
#### output
![Screen Shot 2022-09-05 at 4 18 43 AM](https://user-images.githubusercontent.com/92205707/188436988-0ff81d6f-1ad4-49d7-a2be-36886d2aac1d.png)

-- What about the entire 12 weeks before and after?
-- we use the same queries from previous questions but change it up a bit
-- overall growth rate
```mysql
with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select week_date, week_num, month, year, region, platform, segment, age_band, demographic, customer_type, transactions, sales, 
avg_transaction from weekly_sales_cleaned where year = 2020
and week_num between 12 and 35 order by week_date asc)
select year, week_num, sum(sales) as sales from cte group by year, week_num)
select week_num, year, (case when week_num < 24 then 'Before' else 'After' end) as status, sales from cte1)
select status, year, sum(sales) as total from cte2 group by status, year)
select status, round(100 * total/sum(total) over (partition by year), 1) as percentage_sales, total,
lead(total) over (partition by year) as total1
from cte3)
select status, percentage_sales, total, round(100 * (total - total1)/(total1), 2) as growth_rate from cte4
```
#### output
![Screen Shot 2022-09-05 at 4 19 19 AM](https://user-images.githubusercontent.com/92205707/188437091-8e945557-48d6-4e0b-85e9-75e93d8fa47a.png)

-- How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
-- we use the same queries from previous questions but change it up a bit
-- overall growth rate with 4-week period before and after
```mysql
with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select week_date, week_num, month, year, 
(case when year then 2021 end) as year2021, region, platform, segment, age_band, demographic, 
customer_type, transactions, sales, avg_transaction from weekly_sales_cleaned where week_num between 20 and 27 order by week_date asc)
select year, year2021, week_num, sum(sales) as sales from cte group by year, year2021, week_num)
select week_num, year, year2021, (case when week_num < 24 then 'Before' else 'After' end) as status, sales from cte1)
select status, year, year2021, sum(sales) as total from cte2 group by status, year2021, year)
select status, year, year2021, round(100 * total/sum(total) over (partition by year2021), 1) as percentage_sales, total,
lead(total) over (partition by year) as total1
from cte3)
select status, year, percentage_sales, total, round(100 * (total - total1)/(total1), 2) as growth_rate from cte4
```
#### output
![Screen Shot 2022-09-05 at 4 19 55 AM](https://user-images.githubusercontent.com/92205707/188437185-b43911fd-b53b-4275-b1e5-aa9759015423.png)

