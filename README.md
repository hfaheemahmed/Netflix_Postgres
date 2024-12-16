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
