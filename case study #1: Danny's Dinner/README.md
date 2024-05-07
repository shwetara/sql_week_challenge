# 🍜 Case Study : Danny's Dinner
![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/32a567e0-9263-4bce-a49d-120dd7db0455)

please check this [first](https://8weeksqlchallenge.com/resources/)

# 📚 Table of Contents
- Business Task
- Entity Relationship Diagram
- Questions and Solutions

Danny has shared with you 3 key datasets for this case study:
- sales
- menu
- members

# Business Taks
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

# Entity Relationship Diagram
![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/6f609397-8fb1-40ae-b83f-86db22384dfb)

# Questions and Solutions

### 1. What is the total amount each customer spent at the restaurant?
~~~ sql
select s.customer_id,sum(m.price) as total_price 
from sales as s 
join menu as m  
	on s.product_id = m.product_id
group by customer_id;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/273d7bea-e642-402d-a1b8-755f0f432d18)

### 2.How many days has each customer visited the restaurant?
~~~ sql
select customer_id,count( distinct order_date) as visit_counts
from sales
group by customer_id;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/c795ad78-cbc2-44ca-bca4-9b1302bff024)

### 3.What was the first item from the menu purchased by each customer?
~~~ sql
with cte as (
select s.customer_id,m.product_name, 
dense_rank() over(partition by customer_id order by order_date) as rnk
from sales s
join  menu m 
	on s.product_id = m.product_id
)

select customer_id,product_name
from cte
where cte.rnk =1;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/d502ee4a-8ff4-4536-8514-be77faad205c)

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
~~~ sql
select count(m.product_id) as most_purchased, m.product_name 
from sales s
join menu m on s.product_id= m.product_id
group by m.product_name
order by most_purchased desc
limit 1;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/ccdfbcdb-6fd4-4301-8690-d70f5c8738cc)

### 5.Which item was the most popular for each customer?
~~~ sql
with most_popular as (
select s.customer_id,m.product_name,count(m.product_id) as order_count,
dense_rank() over(partition by s.customer_id order by count(m.product_id) desc) as rnk
from sales s 
join menu m
on s.product_id = m.product_id
group by s.customer_id,m.product_name )

select customer_id,product_name,order_count
from most_popular
where rnk=1;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/28bd1873-c962-4033-a7bd-db0dc64b8774)

### 6.Which item was purchased first by the customer after they became a member?
~~~ sql
with join_members as (
select s.customer_id,s.order_date,s.product_id,
row_number() over(partition by s.customer_id order by s.order_date) as rn
from sales s
join members m
on s.customer_id = m.customer_id and s.order_date > m.join_date)

select jm.customer_id,m.product_name from join_members jm
join menu m on jm.product_id = m.product_id
where rn=1
order by jm.customer_id asc;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/761a6baa-1fe9-4ccf-b2d0-9db674d8f4a1)

### 7.Which item was purchased just before the customer became a member? 
~~~ sql
with before_members as (
select s.customer_id,s.order_date,s.product_id,
row_number() over(partition by m.customer_id order by s.order_date desc) as rn
from sales s
join members m
on s.customer_id = m.customer_id and s.order_date < m.join_date)

select customer_id,m.product_name from before_members bm
join menu m on bm.product_id = m.product_id
where rn=1;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/becd509b-a21f-4035-a80a-8cf89b49020b)

### 8.What is the total items and amount spent for each member before they became a member?
~~~ sql
with members_cte as (
select s.customer_id,s.order_date,s.product_id
from sales s
join members m
on s.customer_id = m.customer_id
and s.order_date < m.join_date)

select customer_id,count(m.product_id) as total_items,sum(m.price) as total_sales
from members_cte mc
join menu m on mc.product_id = m.product_id
group by customer_id 
order by customer_id asc;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/d8396a67-7184-4531-8334-56d9e880a80b)

### 9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
~~~ sql
select s.customer_id,
sum(case when s.product_id=1 then m.price *20 else m.price*10 end) as total_points
from sales s 
join menu m on s.product_id=m.product_id
group by s.customer_id;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/12185fa0-3e52-465d-92f7-bb2ca678bf68)

### 10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
~~~ sql
with date_cte as (
select customer_id,join_date,
date_add(join_date, interval 6 day) as valid_date,last_day('2021-01-01') as last_date
from members)

select s.customer_id,
sum(case when m.product_name ='sushi' then m.price *2*10
		when s.order_date between dc.join_date and dc.valid_date then m.price*2*10
        else m.price*10 end) as total_points  from sales s 
join date_cte dc on s.customer_id = dc.customer_id 
					and dc.join_date <= s.order_date
                    and s.order_date <= dc.last_date
join menu m on m.product_id = s.product_id
group by s.customer_id
order by s.customer_id asc;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/ecafc9b5-abd1-4c63-8412-b4a56bd58bdf)

# Bonus Question
### Join all the underlaying tables using sql
~~~ sql
with cte as (
select s.customer_id,s.order_date,m.product_name,m.price,
(case when s.order_date >= mb.join_date then 'Y'
else 'N' end) as members
from sales s
left join members mb on s.customer_id = mb.customer_id
left join menu m on s.product_id = m.product_id )
select * from cte;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/ae6b271a-d2e5-457e-a9a3-e95815f8b1c0)

### Rank all the things
~~~ sql
with cte as (
select s.customer_id,s.order_date,m.product_name,m.price,
(case when s.order_date >= mb.join_date then 'Y'
else 'N' end) as members
from sales s
left join members mb on s.customer_id = mb.customer_id
left join menu m on s.product_id = m.product_id )



select *,
(case when members='N' then 'null' else dense_rank() over(partition by customer_id,members order by order_date) end) 
as ranking 
from cte;
~~~

![image](https://github.com/shwetara/sql_week_challenge/assets/68580722/cf9c0cff-2180-4ef9-a9d0-68d8d93ef027)

















