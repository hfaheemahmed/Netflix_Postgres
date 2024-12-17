# Netflix_PostgreSQL Project
![NetFlix](https://github.com/hfaheemahmed/Netflix_Postgres/blob/main/netflix.png)

## Querries:
1. Count the number of Movies vs TV Shows
```sql
select type,count(1) from netflix group by type
```
2. Find the most common rating for movies and TV shows
```sql
select type,rating,cnt from (
select type,rating,count(*) cnt,rank() over(partition by type order by count(*) desc) rnk 
from netflix group by type,rating)x where rnk=1
```
3. List all movies released in a specific year (e.g., 2020)
```sql
select title,type,release_year from netflix where release_year=2020 and type='Movie'
```
4. Find the top 5 countries with the most content on Netflix
```sql
select unnest(string_to_array(country,', ')),count(*) from netflix group by 1 order by 2 desc limit 5
```
5. Identify the longest movie
```sql
select title,type,cast(substring(duration,1,position(' ' in duration)) as int) "Duration" from netflix
where duration is not null and type='Movie' order by 3 desc limit 1
```
6. Find content added in the last 5 years
```sql
select title from netflix where to_date(date_added,'Month DD,YYYY') >= current_date - interval '5 year'
```
7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
select title,director from netflix where director ilike '%Rajiv Chilaka%'
```
8. List all TV shows with more than 5 seasons
```sql
select title,type,duration from netflix where type ilike 'tv show'
and split_part(duration,' ',1)::int >5
```
9. Count the number of content items in each genre
```sql
select unnest(string_to_array(listed_in,',')),count(*)  from netflix group by 1
```
10.Find each year and the average numbers of content release in India on netflix. Return top 5 year with highest avg content release!
```sql
select extract(year from to_date(date_added,'Month DD,YYYY')),
round(count(*)::numeric*100/ (select count(*) from netflix where country='India')::numeric,2)
from netflix where country='India' group by 1 order by 1 
```
11. List all movies that are documentaries
```sql
select title,listed_in from netflix where listed_in ilike '%Documentaries%';
```
12. Find all content without a director
```sql
select *  from netflix where director is null
```
13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
select count(*) from  netflix
where  release_year>= extract(year from current_date-interval '10 year')  
and  casts like '%Salman Khan%'
```
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
select count(show_id),trim(unnest(string_to_array(casts,','))) from netflix
where country ilike '%India%' group by 2 order by 1 desc limit 10
```
15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in  the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
```sql
select category,count(*) from (
select case when description ilike '%kill%' or description ilike '%violence%' then 'Bad' 
	   else 'Good' end category from netflix)x
group by 1
```
```
/* --------------------
   8 Week SQL Challenge - Case Study Questions
   --------------------*/
1. What is the total amount each customer spent at the restaurant?
```sql
select customer_id,sum(price) from sales s join menu m on s.product_id=m.product_id
group by customer_id order by 1
```
2. How many days has each customer visited the restaurant?
```sql
select customer_id, count(distinct order_date) NoOfDaysVisted from sales group by customer_id
```
3. What was the first item from the menu purchased by each customer?
```sql
select distinct customer_id,product_name  from sales s join menu m on s.product_id=m.product_id 
 where order_date in (select min(order_date) from sales group by customer_id)
 order by 1

 with cte as (
 select distinct customer_id,product_name,rank() over(partition by customer_id order by order_date) r
 from sales s join menu m on s.product_id=m.product_id )
 select customer_id,product_name from cte where r=1
 order by 1
```
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
 select product_name,count(1) from sales s join menu m on s.product_id=m.product_id
 group by product_name order by count(1) desc limit 1

 select customer_id,count(1) NoofTimes from sales where product_id=(
 select product_id from menu group by product_id order by count(*) desc limit 1)
 group by customer_id
```
5. Which item was the most popular for each customer?
```sql
with cte as (
 select customer_id,product_name,count(1) k from sales s join menu m on s.product_id=m.product_id
 group by customer_id,product_name )
 ,cte1 as (select *,rank() over(partition by customer_id order by k desc)r from cte)
 select customer_id,product_name from cte1 where r=1
```
6. Which item was purchased first by the customer after they became a member?
```sql
with cte as (
 select s.customer_id,m.product_name,s.order_date from sales s  join menu m on s.product_id=m.product_id
  join members on s.customer_id=members.customer_id where s.order_date>members.join_date)
 , cte2 as (select *,rank() over(partition by customer_id order by order_date) r from cte)
 select customer_id,product_name from cte2 where r=1
```
7. Which item was purchased just before the customer became a member?
```sql
with cte as (
 select s.customer_id,m.product_name,s.order_date from sales s  join menu m on s.product_id=m.product_id
  join members on s.customer_id=members.customer_id where s.order_date<members.join_date)
 , cte2 as (select *,dense_rank() over(partition by customer_id order by order_date desc) r from cte)
 select customer_id,product_name from cte2 where r=1
```
8. What is the total items and amount spent for each member before they became a member?
```sql
select mb.customer_id, count(s.product_id),sum(price) 
 from members mb join sales s on mb.customer_id=s.customer_id and s.order_date<mb.join_date
 join menu m on m.product_id=s.product_id
 group by mb.customer_id
```
9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each -- customer have?
```sql
select s.customer_id,
 sum(case when product_name='sushi' then 2*10*price 
          else 10*price end)  points
 from sales s join menu m on s.product_id=m.product_id group by s.customer_id
```
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
select mb.customer_id,
 sum(case when s.order_date between mb.join_date and date_add(mb.join_date,interval '7 day') then price*2*10
      when product_name='sushi' then price*2*10 
     else price*10 end
    ) points
 from members mb join sales s on mb.customer_id=s.customer_id
 join menu m on s.product_id=m.product_id 
 and s.order_date <'2021-02-01'
 group by mb.customer_id
```
11. List all customers,product_name,price and display whether they were member or not on their order date
```sql
select s.customer_id,product_name,price,order_date,join_date,
case when order_date<join_date or join_date is null  then 'No' 
	else 'Yes' end member
```
