### 1. What is the total amount each customer spent at the restaurant?
``` sql
select customer_id, sum(price) total_spent
from sales s 
join menu m 
using(product_id)
group by customer_id
order by 1
``` 
| customer_id | total_spent |
|-------------|-------|
| A           | 76    |
| B           | 74    |
| C           | 36    |
<br> 

### 2. How many days has each customer visited the restaurant?
``` sql
select customer_id, count(distinct order_date) days_visited
from sales
group by customer_id
order by 1
```
| customer_id | days_visited |
|-------------|--------|
| A           | 4      |
| B           | 6      |
| C           | 2      |
<br>

### 3. What was the first item from the menu purchased by each customer?
```sql 
with ct as(
select customer_id, product_name, row_number()over(partition by customer_id order by order_date) latest
from sales s 
join menu m 
using (product_id)
)
select customer_id, product_name 
from ct 
where latest = 1
```
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| B           | curry        |
| C           | ramen        |
<br>

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql
select product_name, count(product_id) as total_buys
from sales s 
	left join menu m 
	using (product_id)
group by product_name
limit 1
```
| product_name | total_buys |
|--------------|------------|
| ramen        | 8          |
<br>

### 5. Which item was the most popular for each customer?
``` sql 
with ct as(
select customer_id, product_name, rank() over(partition by customer_id order by count(*) desc), count(*) times_ordered
from sales s 
	left join menu m 
	using (product_id)
group by customer_id, product_name
order by customer_id )

select customer_id, product_name, times_ordered
from ct 
where rank <2 
``` 
| customer_id | product_name | times_ordered |
|-------------|--------------|---------------|
| A           | ramen        | 3             |
| B           | sushi        | 2             |
| B           | curry        | 2             |
| B           | ramen        | 2             |
| C           | ramen        | 3             |
<br>

### 6. Which item was purchased first by the customer after they became a member?
``` sql 
with rn as (
select customer_id customer, join_date,order_date, product_name product, row_number() over(partition by customer_id order by order_date) row
from dannys_diner.sales s 
	left join members m 
	using(customer_id)
	left join menu me 
	on s.product_id = me.product_id
where order_date >= join_date
)

select customer, product, join_date, order_date
from rn 
where row = 1
```
| customer_id | product_name | order_date | join_date  |
|-------------|--------------|------------|------------|
| A           | curry        | 2021-01-07 | 2021-01-07 |
| B           | sushi        | 2021-01-11 | 2021-01-09 |
<br>

### 7. Which item was purchased just before the customer became a member?
``` sql 
with rn as (
select customer_id customer, join_date,order_date, product_name product, row_number() over(partition by customer_id order by order_date desc) row
from dannys_diner.sales s 
	inner join members m 
	using(customer_id)
	left join menu me 
	on s.product_id = me.product_id
where order_date < join_date 
	)
	
select customer, product, join_date, order_date
from rn 
where row = 1
```
| customer | product | order_date | join_date  |
|----------|---------|------------|------------|
| A        | sushi   | 2021-01-01 | 2021-01-07 |
| B        | sushi   | 2021-01-04 | 2021-01-09 |
<br>

### 8. What is the total items and amount spent for each member before they became a member?
``` sql 
with tot as (
select *
from dannys_diner.sales s 
	inner join members m 
	using(customer_id)
	left join menu me 
	on s.product_id = me.product_id
where order_date < join_date 
) 

select customer_id, count(*) items_bought, sum(price) amount_spent 
from tot
group by customer_id
order by 1
```
| customer_id | items_bought | amount_spent |
|-------------|--------------|--------------|
| A           | 2            | 25           |
| B           | 3            | 40           |
<br>

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
``` sql 
with po as (
select customer_id, product_name, price, 
	(case when product_name = 'sushi' THEN price * 20 ELSE price * 10 END) points 
from sales s 
	left join menu m 
	using (product_id)
	) 
	
select customer_id, sum(points) points
from po 
group by 1
order by 1 
```
| customer_id | points       |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January? 
``` sql 
/* Calculate points when the order date is in the first week after joining */ 
with fw as (
select *, (price * 20) points
from dannys_diner.sales s 
	inner join members m 
	using(customer_id)
	left join menu me 
	on s.product_id = me.product_id
where order_date >= join_date and order_date <= (join_date + interval '6 days')
	), 

/* Calculate points when the order date is not in the first week after joining */ 
ow as(
select *, (case when product_name = 'sushi' then price * 20 else price * 10 end) points
from dannys_diner.sales s 
	inner join members m 
	using(customer_id)
	left join menu me 
	on s.product_id = me.product_id
where order_date NOT IN (
		select order_date 
		from sales s 
		join members m 
		using(customer_id)
where order_date >= join_date and order_date <= (join_date + interval '6 days')
	)
/* combine both cte's using union to stack the rows on each other. This is the final cte */
),
final as(
	select *
	from fw
	union all
	select *
	from ow 
	where order_date between '2021-01-01' and '2021-01-31'
)

select customer_id customer, sum(points) total_points
from final 
group by 1
order by 1
```
| customer    | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |
<br>
