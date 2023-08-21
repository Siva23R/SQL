# SQL
SQL projects
#what is the toatl amunt each customer spent on zomato
select s.userid,sum(p.price) as total_price
from sales s join product p
on s.product_id = p.product_id
group by s.userid;

#how many days each customer visited zomato
select userid, count(created_date) as visited_time
from sales
group by userid;

# first product purchsed by each customer
select* from(
select *,rank() over ( partition by userid order by created_date) as rnk
from sales) X
where X.rnk =1;

#what are the most purchased item on menu and how many times was it purchase  by all customer
select userid,count(product_id) as times
from sales
where product_id =
(select 
p.product_id
from sales s join product p
on s.product_id = p.product_id
group by product_id
limit 1)
group by userid;

#which of the item is most popular for each of the customers
select userid,product_id
from(select userid,product_id,count(product_id),rank() over (partition by userid order by count(product_id) desc) as rnk
from sales
group by userid,product_id) X
where X.rnk = 1;

#which item was brougth 1st by a customer after become member
select userid,created_date,product_id from(select*, rank() over (partition by userid order by created_date) as rnk
from(select s.userid,s.created_date,s.product_id,g.gold_signup_date
from sales s
join goldusers_signup g on s.userid =g.userid and s.created_date > g.gold_signup_date) X)y
where y.rnk =1;

#which item was purchased customer just before become a member
select * from(select*, rank() over (partition by userid order by created_date desc) as rnk
from(select s.userid,s.created_date,s.product_id,g.gold_signup_date
from sales s
join goldusers_signup g on s.userid =g.userid and s.created_date < g.gold_signup_date) X)y
where y.rnk =1;

#what is the total order and amount spent for  each member before they become member
with t1 as(
select s.userid,s.created_date,s.product_id,p.price,g.gold_signup_date
from sales s 
join goldusers_signup g on s.userid = g.userid and s.created_date < g.gold_signup_date
join product p  on s.product_id = p.product_id)
select userid,count(product_id) as tot_orders,sum(price) as tot_amount
from t1
group by userid
order by userid;

#If buying each prodcut genrates zomoto point,  for example 5rs = 2points for example p1 5rs = 1point, p2 10rs = 5point , p3 = 5rs = 1point
#Calculated the points collected by each customers and which product most points have benn given
with t1 as (
select s.userid,p.product_name,sum(p.price) as total,
case when product_name = "p1" then "5"
	when product_name = "p2" then "2"
    when product_name = "p3" then "5"
    else product_name
    end as div_points
from sales s
join product p on s.product_id = p.product_id
group by s.userid,p.product_name
order by userid),
t2 as(
select *,(total/div_points)*2.5 as total_zomoto_point_amount
from t1)
select userid,sum(total_zomoto_point_amount) as total_amount
from t2
group by userid
order by total_amount desc;

with t1 as (
select s.userid,p.product_name,sum(p.price) as total,
case when product_name = "p1" then "5"
	when product_name = "p2" then "2"
    when product_name = "p3" then "5"
    else product_name
    end as div_points
from sales s
join product p on s.product_id = p.product_id
group by s.userid,p.product_name
order by userid),
t2 as(
select *,(total/div_points)*2.5 as total_zomoto_point_amount
from t1),
t3 as (
select product_name,sum(total_zomoto_point_amount) as total_amount,rank() over (order by sum(total_zomoto_point_amount )desc)rnk
from t2
group by product_name)
select *
from t3 
where rnk = 1; 

#In the first one year after customer joins the gold programs(including the join date) irrespective of what the customer has purchased they earn 5 zomoto pints fro every 10 rs sprnt who earned more 1 or 3 and
#what was the pont earing in their 1st year

select s.userid,s.product_id,s.created_date,g.gold_signup_date,p.price,round((p.price/2),0) as zomoto_points 
from sales s
join goldusers_signup g on s.userid = g.userid and s.created_date >= g.gold_signup_date and s.created_date <= date_add(g.gold_signup_date,interval 1 year)
join product p on s.product_id = p.product_id
group by s.userid,s.product_id,s.created_date,g.gold_signup_date,p.price
order by userid ;

select *
from(select *,rank() over (order by  zomoto_points desc) as rnk
from(select s.userid,s.product_id,s.created_date,g.gold_signup_date,p.price,round((p.price/2),0) as zomoto_points 
from sales s
join goldusers_signup g on s.userid = g.userid and s.created_date >= g.gold_signup_date and s.created_date <= date_add(g.gold_signup_date,interval 1 year)
join product p on s.product_id = p.product_id
group by s.userid,s.product_id,s.created_date,g.gold_signup_date,p.price
order by userid) X) Y
where Y.rnk = 1;

# rank all transaction of customers
select*, dense_rank() over ( partition by userid order by created_date) as rnk
from sales ;

# rank all the transaction for each member whenever they are a zomoto gold member for every non gold member transaction mark as na

with t1 as(
select s.userid,s.created_date,g.gold_signup_date
from sales s
left join goldusers_signup g on s.userid = g.userid and s.created_date >= g.gold_signup_date)
select *, case when gold_signup_date is null then "NA" else rank() over (partition by userid order by s.created_date desc) end as rnk 
from t1;
