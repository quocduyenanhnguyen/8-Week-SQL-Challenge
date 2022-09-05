```mysql
use Dinner
```

### 1. What is the total amount each customer spent at the restaurant?
```mysql
select s.customer_id, sum(m.price) from sales as s
join menu as m on m.product_id = s.product_id
group by s.customer_id
```
#### output

![Screen Shot 2022-09-05 at 2 26 03 AM](https://user-images.githubusercontent.com/92205707/188416318-a8ad2a89-a902-4adc-bf55-76e267f64ca1.png)

### 2. How many days has each customer visited the restaurant?
```mysql
select customer_id, count(distinct order_date) as num_of_days_visited from sales
group by customer_id
```
#### output
![Screen Shot 2022-09-05 at 2 27 40 AM](https://user-images.githubusercontent.com/92205707/188416660-3f98a835-1b2a-4c8b-86ed-830cf48bd372.png)

### 3. What was the first item from the menu purchased by each customer?
```mysql
select distinct s.customer_id, m.product_name as 'fist time purchase', s.order_date,
dense_rank() over (PARTITION BY s.customer_id order by s.order_date) first_time_purchase from sales as s
join menu as m on m.product_id = s.product_id
order by first_time_purchase
limit 4
```
#### output
![Screen Shot 2022-09-05 at 2 28 42 AM](https://user-images.githubusercontent.com/92205707/188416889-8f785bef-d6aa-457c-87ab-045b128a0c03.png)

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```mysql
select m.product_name as 'most purchased item', count(s.product_id) as 'purchase frequency' from sales as s
join menu as m on m.product_id = s.product_id
group by m.product_name
order by count(s.product_id) desc
limit 1
```
#### output
![Screen Shot 2022-09-05 at 2 29 50 AM](https://user-images.githubusercontent.com/92205707/188417141-cf2027bb-f939-4ca5-b0f9-2028226c12ef.png)

### 5. Which item was the most popular for each customer?
```mysql
create view rank_food as select distinct s.customer_id, m.product_name as most_purchased_item_by_customer,
count(m.product_name),
dense_rank() over (partition by s.customer_id order by count(s.customer_id) desc)
item_rank from sales as s
join menu as m on m.product_id = s.product_id
group by s.customer_id, m.product_name
order by item_rank desc
```
```mysql
select * from rank_food where item_rank = 1
```
#### output
![Screen Shot 2022-09-05 at 2 31 33 AM](https://user-images.githubusercontent.com/92205707/188417509-e1247ab8-df3b-4eed-8373-0645ceee9540.png)

### 6. Which item was purchased first by the customer after they became a member?
```mysql
create view after_member as select distinct s.customer_id, menu.product_name, s.order_date,
dense_rank () over (partition by s.customer_id order by s.order_date asc) as rank_member
from sales as s
join members as m on m.customer_id = s.customer_id
join menu on menu.product_id = s.product_id
where s.order_date >= m.join_date
order by s.order_date asc
```
```
select * from after_member where rank_member = 1
```
#### output
![Screen Shot 2022-09-05 at 2 32 39 AM](https://user-images.githubusercontent.com/92205707/188417719-2abc3a8b-8b0f-445e-8f29-63e6bb3ae648.png)

### 7. Which item was purchased just before the customer became a member?
```mysql
select distinct s.customer_id, menu.product_name, s.order_date,
dense_rank () over (partition by s.customer_id order by s.order_date desc) as rank_member
from sales as s
join members as m on m.customer_id = s.customer_id
join menu on menu.product_id = s.product_id
where s.order_date < m.join_date
limit 3
```
![Screen Shot 2022-09-05 at 2 33 16 AM](https://user-images.githubusercontent.com/92205707/188417837-70465a59-5451-4e2d-ac29-3eb4097453a5.png)

### 8. What is the total items and amount spent for each member before they became a member?
```mysql
select s.customer_id, count(menu.product_name) as total_items,
sum(menu.price) as total_spent
from sales as s
join members as m on m.customer_id = s.customer_id
join menu on menu.product_id = s.product_id
where s.order_date < m.join_date
group by s.customer_id
```
#### output
![Screen Shot 2022-09-05 at 2 33 59 AM](https://user-images.githubusercontent.com/92205707/188417974-2c9af324-586a-4f00-a2f6-423ba2afe6a9.png)

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

#### 1st version
``` mysql
create view points_accumulated as select s.customer_id, 
sum(case when menu.product_name = 'sushi' then (menu.price * 20) else 0 end) as sushi_points,
sum(case when menu.product_name = 'curry' then (menu.price * 10) else 0 end) as curry_points,
sum(case when menu.product_name = 'ramen' then (menu.price * 10) else 0 end) as ramen_points
from sales as s
join menu on menu.product_id = s.product_id
group by s.customer_id
```
```
select customer_id, sum(sushi_points + curry_points + ramen_points) as total_points from points_accumulated group by customer_id
```
#### output
![Screen Shot 2022-09-05 at 2 34 59 AM](https://user-images.githubusercontent.com/92205707/188418145-ab54ec95-3287-48e9-bbeb-a5cb175adf67.png)

#### 2nd version
```
with cte as (select s.customer_id, 
sum(case when menu.product_name = 'sushi' then (menu.price * 20) else 0 end) as sushi_points,
sum(case when menu.product_name = 'curry' then (menu.price * 10) else 0 end) as curry_points,
sum(case when menu.product_name = 'ramen' then (menu.price * 10) else 0 end) as ramen_points
from sales as s
join menu on menu.product_id = s.product_id
group by s.customer_id)
select customer_id, sum(sushi_points + curry_points + ramen_points) from cte group by customer_id
```
#### output
![Screen Shot 2022-09-05 at 2 36 53 AM](https://user-images.githubusercontent.com/92205707/188418528-0c64a129-84be-4933-b6ce-4a5fbfd2cea4.png)

#### 3rd version
```
select s.customer_id, 
sum(case when menu.product_name = 'sushi' then (menu.price * 20)
when menu.product_name = 'curry' then (menu.price * 10) 
when menu.product_name = 'ramen' then (menu.price * 10) else 0 end) as points
from sales as s
join menu on menu.product_id = s.product_id
group by s.customer_id
```
#### output
![Screen Shot 2022-09-05 at 2 37 41 AM](https://user-images.githubusercontent.com/92205707/188418674-b6724edf-6ee6-4661-b97d-8728f4c6c713.png)

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```mysql
create view dates_cte as SELECT *, DATE_ADD(join_date, interval 6 day) AS valid_date, last_day('2021-01-1') AS last_date FROM members AS m

SELECT d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price,
SUM(CASE 
WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price 
WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price ELSE 10 * m.price END) AS points
FROM dates_cte AS d
JOIN sales AS s ON d.customer_id = s.customer_id
JOIN menu AS m ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price
order by customer_id asc
```
#### output
![Screen Shot 2022-09-05 at 2 40 00 AM](https://user-images.githubusercontent.com/92205707/188419166-f473c6ea-69ec-4089-96fe-1028d3a6b005.png)

### Bonus question
#### join all the things
```mysql
select s.customer_id, s.order_date, menu.product_name, menu.price, 
(case when s.order_date >= m.join_date then 'Y' else 'N' end) as member
from sales as s
join menu on menu.product_id = s.product_id 
join members as m on m.customer_id = s.customer_id
order by s.customer_id, s.order_date asc
```
#### output
![Screen Shot 2022-09-05 at 2 42 18 AM](https://user-images.githubusercontent.com/92205707/188419644-61182ee1-7edf-4941-bd8e-b70edf965c29.png)

#### rank all the things
```mysql
with cte as (select s.customer_id, s.order_date, m.join_date, menu.product_name, menu.price, 
case when s.order_date >= m.join_date then 'Y' else 'N' end as member
from sales as s
join menu on menu.product_id = s.product_id 
join members as m on m.customer_id = s.customer_id
order by s.customer_id, s.order_date asc) 
select customer_id, order_date, product_name, price,
(case when member = 'N' then 'null' else dense_rank() over (partition by customer_id, member order by order_date) end) as ranking
from cte
```
#### output
![Screen Shot 2022-09-05 at 2 43 09 AM](https://user-images.githubusercontent.com/92205707/188419799-5fc08556-7187-4ba4-af43-8e4f66eb110a.png)
