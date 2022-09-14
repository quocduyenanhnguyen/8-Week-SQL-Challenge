```mysql
use fresh_segments
```
### Data Exploration and Cleansing

#### Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
```mysql
create view interest_metrics1 as with cte as (SELECT month, _year, concat(month_year, '-01') as 'date', interest_id, composition, index_value, ranking, percentile_ranking
FROM interest_metrics)
select month, _year, STR_TO_DATE(date,'%m-%Y-%d') as date, interest_id, composition, index_value, ranking, percentile_ranking
from cte
```
#### output
![Screen Shot 2022-09-14 at 2 39 06 AM](https://user-images.githubusercontent.com/92205707/190119565-bbfba75e-1c40-4372-a9cb-b629d5045541.png)

### What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```mysql
select date, count(*) as count from interest_metrics1 group by date order by date is null desc
```
#### output
![Screen Shot 2022-09-14 at 2 39 58 AM](https://user-images.githubusercontent.com/92205707/190119760-6e39a1d5-d515-4874-bd3c-0291c1f7b672.png)

#### How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
```mysql
select interest_id as 'id that is not in map table', count(distinct interest_id) from interest_metrics1 where interest_id not in (select distinct id from interest_map)
group by interest_id;

select distinct id as 'id that is not in metrics table', count(distinct id) from interest_map where id not in (select distinct interest_id from interest_metrics1)
group by id
```
#### output
![Screen Shot 2022-09-14 at 2 40 47 AM](https://user-images.githubusercontent.com/92205707/190119928-5714cca9-902b-493b-a817-d36f16335ad3.png)
![Screen Shot 2022-09-14 at 2 41 01 AM](https://user-images.githubusercontent.com/92205707/190119983-a3a10bd9-1ff8-4a1f-966e-e6f584af0826.png)

#### Summarise the id values in the fresh_segments.interest_map by its total record count in this table
```mysql
select distinct m.interest_name, count(i.interest_id) as count from interest_metrics1 as i 
join interest_map as m on m.id = i.interest_id
group by m.interest_name, i.interest_id order by count desc
```
#### output
![Screen Shot 2022-09-14 at 2 41 47 AM](https://user-images.githubusercontent.com/92205707/190120149-76b76eb3-64e0-4d63-a831-44ab7f4a771d.png)

#### What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.

-- we use join and join on interest_id field 
```mysql
select distinct m.interest_name, m.interest_summary, m.created_at, m.last_modified, i.month, i._year, i.date,
i.composition, i.index_value, i.ranking, i.percentile_ranking
from interest_metrics1 as i 
join interest_map as m on m.id = i.interest_id
where i.interest_id = 21246
```
#### output
![Screen Shot 2022-09-14 at 2 42 51 AM](https://user-images.githubusercontent.com/92205707/190120408-776a9ff2-9780-4179-8157-ba1965e76e1e.png)

#### Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

-- We have 188 records in which the month_year value is before the created_at value, we will double check to see if they are in the same month
```mysql
with cte as (select distinct m.interest_name, m.interest_summary, m.created_at, m.last_modified, i.month, i._year, i.date,
i.composition, i.index_value, i.ranking, i.percentile_ranking
from interest_metrics1 as i 
join interest_map as m on m.id = i.interest_id)
select interest_name, interest_summary, created_at, last_modified, month, _year, date as month_year, composition, index_value, ranking, 
percentile_ranking from cte where date < created_at
```
#### output
![Screen Shot 2022-09-14 at 2 43 42 AM](https://user-images.githubusercontent.com/92205707/190120632-e311de4d-a323-4702-84cf-314882381830.png)

-- it seems these records are in the same month, so we will consider them as valid
```mysql
with cte1 as (with cte as (select distinct m.interest_name, m.interest_summary, m.created_at, m.last_modified, i.month, i._year, i.date,
i.composition, i.index_value, i.ranking, i.percentile_ranking
from interest_metrics1 as i 
join interest_map as m on m.id = i.interest_id)
select interest_name, interest_summary, created_at, last_modified, month, _year, date as month_year, month(date) as month1, composition, index_value, ranking, 
percentile_ranking from cte where date < created_at)
select * from cte1 where month <> month1
```
#### output
![Screen Shot 2022-09-14 at 2 44 32 AM](https://user-images.githubusercontent.com/92205707/190120820-41b9386d-9687-4403-b30c-bc98a08b73cf.png)

### Interest Analysis

#### Which interests have been present in all month_year dates in our dataset?
-- Since we have 14 months, we will filter to find which interests are present in all 14 months by counting the total number of occurences in all months, if it is 14 then we are sure that they are present in all month_year
```mysql
with cte as (select distinct interest_id, count(distinct date) as count from interest_metrics1 group by interest_id order by count desc)
select * from cte where count = 14 order by interest_id asc
```
#### output
![Screen Shot 2022-09-14 at 2 47 46 AM](https://user-images.githubusercontent.com/92205707/190121573-8ed175b2-417e-44c8-a1c7-3c7b1bfe2786.png)


-- There are 480 interests that are present in all months
```mysql
with cte1 as (with cte as (select distinct interest_id, count(distinct date) as count from interest_metrics1 group by interest_id order by count desc)
select * from cte where count = 14 order by interest_id asc)
select count(*) from cte1
```
#### output
![Screen Shot 2022-09-14 at 2 48 08 AM](https://user-images.githubusercontent.com/92205707/190121663-63866cb4-8c88-4a30-93ba-74c0db34dcc8.png)

#### Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
```mysql
with cte3 as (with cte2 as (with cte1 as (with cte as (select distinct interest_id, count(distinct date) as num_of_months from interest_metrics1 group by interest_id order by num_of_months desc)
select num_of_months, count(interest_id) as count from cte group by num_of_months)
select num_of_months, count, sum(count) over (order by num_of_months desc) as cumulative_sum from cte1) 
select *, round((cumulative_sum * 100)/(select sum(count) from cte2), 1) as cumulative_percentage from cte2)
select * from cte3 where cumulative_percentage > 90
```
#### output
![Screen Shot 2022-09-14 at 2 48 50 AM](https://user-images.githubusercontent.com/92205707/190121840-c23c66ec-c6c7-41fb-bcdc-c15d6e2a64fc.png)

### Segment Analysis

#### Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
-- top 10
```mysql
with cte2 as (with cte1 as (with cte as (select distinct interest_id, count(distinct date) as count from interest_metrics1
group by interest_id order by count desc)
select distinct cte.interest_id, cte.count, i.date, i.composition, i.index_value, i.ranking, i.percentile_ranking
from cte 
join interest_metrics1 as i on i.interest_id = cte.interest_id
where count >= 6)
select interest_id, max(composition) as largest_composition from cte1
group by interest_id
order by largest_composition desc
limit 10)
select cte2.interest_id, i.date, cte2.largest_composition from cte2
join interest_metrics1 as i on i.interest_id = cte2.interest_id and cte2.largest_composition = i.composition
order by largest_composition desc;
```
#### output
![Screen Shot 2022-09-14 at 2 49 47 AM](https://user-images.githubusercontent.com/92205707/190122043-f21edfbe-58e3-4ebd-8836-a23bf342c978.png)

-- bottom 10
```mysql
with cte2 as (with cte1 as (with cte as (select distinct interest_id, count(distinct date) as count from interest_metrics1
group by interest_id order by count desc)
select distinct cte.interest_id, cte.count, i.date, i.composition, i.index_value, i.ranking, i.percentile_ranking
from cte 
join interest_metrics1 as i on i.interest_id = cte.interest_id
where count >= 6)
select interest_id, max(composition) as smallest_composition from cte1
group by interest_id
order by smallest_composition 
limit 10)
select cte2.interest_id, i.date, cte2.smallest_composition from cte2
join interest_metrics1 as i on i.interest_id = cte2.interest_id and cte2.smallest_composition = i.composition
order by smallest_composition asc;
```
#### output
![Screen Shot 2022-09-14 at 2 50 23 AM](https://user-images.githubusercontent.com/92205707/190122173-559cd49d-5311-4efc-9ed3-8ce4b14db09a.png)

#### Which 5 interests had the lowest average ranking value?
```mysql
select interest_id, avg(ranking) as average_ranking from interest_metrics1 group by interest_id order by average_ranking asc limit 5
```
#### output
![Screen Shot 2022-09-14 at 2 50 59 AM](https://user-images.githubusercontent.com/92205707/190122339-9fb9d1c4-181d-4fac-9fbb-91069cb92bc5.png)

#### Which 5 interests had the largest standard deviation in their percentile_ranking value?
```mysql
select interest_id, round(STDDEV_SAMP(percentile_ranking), 1) as standard_deviation from interest_metrics1 group by interest_id
order by standard_deviation desc limit 5;
```
#### output
![Screen Shot 2022-09-14 at 2 51 40 AM](https://user-images.githubusercontent.com/92205707/190122499-52d28863-4433-4b5d-9fb6-d01e74b84952.png)

#### For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
```mysql
with cte1 as (with cte as (select interest_id, round(STDDEV_SAMP(percentile_ranking), 1) as standard_deviation from interest_metrics1 group by interest_id
order by standard_deviation desc limit 5)
select distinct cte.interest_id, cte.standard_deviation, min(i.percentile_ranking) as minimum_value from cte
join interest_metrics1 as i on i.interest_id = cte.interest_id
group by cte.interest_id, cte.standard_deviation)
select distinct cte1.interest_id, cte1.standard_deviation, cte1.minimum_value, i.date from cte1
join interest_metrics1 as i on i.interest_id = cte1.interest_id and cte1.minimum_value = i.percentile_ranking
order by standard_deviation desc;

with cte1 as (with cte as (select interest_id, round(STDDEV_SAMP(percentile_ranking), 1) as standard_deviation from interest_metrics1 group by interest_id
order by standard_deviation desc limit 5)
select distinct cte.interest_id, cte.standard_deviation, max(i.percentile_ranking) as maximum_value from cte
join interest_metrics1 as i on i.interest_id = cte.interest_id
group by cte.interest_id, cte.standard_deviation)
select distinct cte1.interest_id, cte1.standard_deviation, cte1.maximum_value, i.date from cte1
join interest_metrics1 as i on i.interest_id = cte1.interest_id and cte1.maximum_value = i.percentile_ranking
order by standard_deviation desc
```
#### output
![Screen Shot 2022-09-14 at 2 52 31 AM](https://user-images.githubusercontent.com/92205707/190122720-052b120f-15b5-4893-a8ab-4984caca0842.png)
![Screen Shot 2022-09-14 at 2 52 43 AM](https://user-images.githubusercontent.com/92205707/190122762-36d80a70-d817-4e9e-b406-8463e87c1520.png)

#### Index Analysis

-- The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients. Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

#### What is the top 10 interests by the average composition for each month?
```mysql
with cte1 as (with cte as (select interest_id, month, _year, round(composition/index_value, 2) as average_comp from interest_metrics1)
select *, dense_rank() over (partition by month, _year order by average_comp desc) as index_rank from cte)
select m.interest_name, cte1.month, cte1._year, cte1.average_comp, cte1.index_rank from cte1 
join interest_map as m on m.id = cte1.interest_id
where cte1.index_rank <= 10;
```
#### output
![Screen Shot 2022-09-14 at 2 53 45 AM](https://user-images.githubusercontent.com/92205707/190122985-916aaaa7-f3a9-40bd-a8e7-ebeaa3c2122f.png)

#### For all of these top 10 interests - which interest appears the most often?
```mysql
with cte2 as (with cte1 as (with cte as (select interest_id, month, _year, round(composition/index_value, 2) as average_comp from interest_metrics1)
select *, dense_rank() over (partition by month, _year order by average_comp desc) as index_rank from cte)
select m.interest_name, cte1.month, cte1._year, cte1.average_comp, cte1.index_rank from cte1 
join interest_map as m on m.id = cte1.interest_id
where cte1.index_rank <= 10)
select interest_name, count(interest_name) as count from cte2 group by interest_name order by count desc
```
#### output
![Screen Shot 2022-09-14 at 2 54 24 AM](https://user-images.githubusercontent.com/92205707/190123125-d6b5bb2b-d852-40fa-8d45-053551b484fb.png)

#### What is the average of the average composition for the top 10 interests for each month?
```mysql
with cte2 as (with cte1 as (with cte as (select interest_id, date, round(composition/index_value, 2) as average_comp from interest_metrics1)
select *, dense_rank() over (partition by date order by average_comp desc) as index_rank from cte)
select m.interest_name, cte1.date, cte1.average_comp, cte1.index_rank from cte1 
join interest_map as m on m.id = cte1.interest_id
where cte1.index_rank <= 10)
select date, round(avg(average_comp), 2) as average from cte2 group by date order by 1 asc
```
#### output
![Screen Shot 2022-09-14 at 2 55 07 AM](https://user-images.githubusercontent.com/92205707/190123296-483017ff-2faf-46c4-b8bc-c2cc85dbfb64.png)

#### What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
```mysql
with cte4 as (with cte3 as (with cte2 as (with cte1 as (with cte as (select interest_id, date, round(composition/index_value, 2) as average_comp from interest_metrics1)
select *, dense_rank() over (partition by date order by average_comp desc) as index_rank from cte)
select m.interest_name, cte1.date, cte1.average_comp, cte1.index_rank,
concat(m.interest_name,': ', cte1.average_comp) as interest_name1
from cte1 
join interest_map as m on m.id = cte1.interest_id
where cte1.index_rank <= 10)
select distinct first_value(interest_name) over (partition by date order by date asc) as interest_name, date,
max(average_comp) over (partition by date order by date) as max_index_composition,
first_value(interest_name1) over (partition by date order by date) as demo from cte2)
select distinct interest_name, date, max_index_composition, round(avg(max_index_composition) over (order by date rows between 2 preceding and 0 following), 2) 
as 3_month_moving_avg, lag(demo) over (order by date, interest_name) as 1_month_ago,
lag(demo, 2) over (order by date, interest_name) as 2_months_ago
from cte3)
select * from cte4 where 1_month_ago is not null and 2_months_ago is not null
```
#### output
![Screen Shot 2022-09-14 at 2 55 50 AM](https://user-images.githubusercontent.com/92205707/190123445-d64ffc16-f32d-476b-82d1-68ee8f756d66.png)


