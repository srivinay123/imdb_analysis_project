# imdb_analysis_project

use imdb_project_sql;

  #########SEGMENT--1 -########## Database - Tables, Columns, Relationships
Q 1.1 What are the different tables in the database and how are they connected to each other in the database?

SELECT * FROM DIRECTOR_MAPPING;
SELECT * FROM genre;
SELECT * FROM MOVIES;
SELECT * FROM names;
SELECT * FROM ratings;
SELECT * FROM ROLE_MAPPING;

Q 1.2 Find the total number of rows in each table of the schema.

select count(*) from director_mapping;
select count(*) from genre;
select count(*) from movie;
select count(*) from names;
select count(*) from ratings;
select count(*) from role_mapping;

Q 1.3 Identify which columns in the movie table have null values.

SELECT 
	(select count(*) from movie where id is NULL) as id_null,
	(select count(*) from movie where title is NULL) as title_null,
	(select count(*) from movie where year is NULL) as year_null,
	(select count(*) from movie where date_published is NULL) as date_published_null,
	(select count(*) from movie where duration is NULL) as duration_null,
	(select count(*) from movie where country is NULL) as country_null,
	(select count(*) from movie where worlwide_gross_income is NULL) as worldwide_gross_income_null,
	(select count(*) from movie where languages is NULL) as laguages_null,
	(select count(*) from movie where production_company is NULL) as production_company_null
;

  #########SEGMENT--2 -########## Movie Release Trends
Q 2.1 Determine the total number of movies released each year and analyse the month-wise trend.

select year, count(id) movie_count from movie
group by year
order by 1;

select month(date_published) monthwise, count(id) movie_count from movie
group by 1
order by 2 desc; 

Q 2.2 Calculate the number of movies produced in the USA or India in the year 2019.

select count(*) movie_count from movie
where year = '2019' and (lower(country) like '%usa%' or lower(country) like '%india%');

  #########SEGMENT--3 -########## Production Statistics and Genre Analysis
Q 3.1 Retrieve the unique list of genres present in the dataset.

select distinct genre as No_of_genre from genre;

Q 3.2 Identify the genre with the highest number of movies produced overall.

select genre, count(movie_id) from genre
group by genre
order by 2 desc
limit 1;

Q 3.3 Determine the count of movies that belong to only one genre.

select count(*) from (select movie_id, count(genre) from genre
group by movie_id
having count(genre) = 1) sub

Q 3.4 Calculate the average duration of movies in each genre.

select genre, round(avg(duration),2) Average_duration from genre g
join movie m on m.id = g.movie_id
group by genre
order by 2 desc;

Q 3.5 Find the rank of the 'thriller' genre among all genres in terms of the number of movies produced.

select * from (select genre, Dense_rank() over(order by count(movie_id) desc) ranking from genre
group by 1) subquery where genre = 'thriller';


  #########SEGMENT--4 -########## Ratings Analysis and Crew Members
Q 4.1 Retrieve the minimum and maximum values in each column of the ratings table (except movie_id).

select max(avg_rating) as maximum_avg_rating, min(avg_rating) as minimum_avg_rating, 
       max(total_votes) as maximum_total_votes, min(total_votes) as minimum_total_votes,
       max(median_rating) as maximum_median_rating, min(median_rating) as minimum_median_rating
from ratings;

Q 4.2 Identify the top 10 movies based on average rating.

select title as Movie_Name, avg_rating from movie m
join ratings r on r.movie_id = m.id
order by 2 desc 
limit 10;

Q 4.3 Summarise the ratings table based on movie counts by median ratings.

SELECT
    median_rating,
    COUNT(*) AS movie_count
FROM ratings
GROUP BY median_rating
ORDER BY median_rating;


Q 4.4 Identify the production house that has produced the most number of hit movies (average rating > 8).

select production_company, count(id) number_of_movies from movies
where id in (select movie_id from ratings where avg_rating > 8)
group by 1
order by 2 desc;

Q 4.5 Determine the number of movies released in each genre during March 2017 in the USA with more than 1,000 votes.

select genre, count(g.movie_id) movie_count from genre g
join movies m on m.id = g.movie_id
join ratings r on g.movie_id = r.movie_id
where total_votes > 1000 and lower(country) like'%usa%' and year = 2017 and month(date_published) = 3
group by 1
order by 2 desc;

SELECT g.genre, COUNT(*) AS movie_count
FROM movies m
INNER JOIN movies m ON m.id = g.movie_id
INNER JOIN ratings r ON r.movie_id = g.movie_id
WHERE  year - 2017 and month(date_published) = 3 and lower(country) like"%usa" and 
total_votes  > 1000
GROUP BY g.genre
ORDER BY movie_count DESC;

Q 4.6 Retrieve movies of each genre starting with the word 'The' and having an average rating > 8.

select genre, title, avg_rating as no_of_movies from genre g
join movies m on g.movie_id = m.id 
join ratings r on r.movie_id = g.movie_id 
where title like 'the%' and avg_rating > 8
order by 1;

################SEGMENT--5###############  Crew Analysis

Q 5.1 -	Identify the columns in the names table that have null values.
select count(*) as id_null from names where id is null;
select count(*)as name_null from names where name is null;
select count(*)as height_null from names where height is null;
select count(*) as date_of_birth_null from names where date_of-birth is null;
select count(* ) as known_for_movies from names where known_for_movies is null;


Q 5.2 -	Determine  top three directors in the top three genres with movies having an average rating > 8.

with top_genres as
(select genre from genre g  
inner join ratings r on r.movie_id = g.movie_id 
where avg_rating >8  group by genre order by count( g.movie_id) desc limit 3)

select n.name as top_director, count(n.id) as  movie_count
from names n 
inner join role_mapping rm on rm.namE_ID =n.id
inner join director_mapping dm on rm.movie_id=dm.movie_id
inner join movies m on m.id = dm.movie_id 
inner join genre g on g.movie_id = m.id 
inner join ratings r on r.movie_id = m.id
where avg_rating >8
 group by 1 order by movie_count desc  limit 3;
 

 
Q 5.3 -	Find the top two actors whose movies have a median rating >= 8
 
select  n.name as top_actor,count(m.id) as movie_count  from  names n 
inner join role_mapping rm on rm.name_id = n.id 
inner join movies m on  m.id=rm.movie_id
inner join ratings r on r.movie_id=m.id
where r.median_rating >= 8
and  lower(category) like '%actor%'
group by top_actor order by movie_count desc limit 2;



Q 5.4   -	Identify the top three production houses based on the number of votes received by their movies

select production_company as top_production, sum(total_votes) as votes from movies m
 inner join ratings r on r.movie_id = m.id
 where production_company is not null
 group by 1 order by votes desc limit 3;
 
 
 Q 5.5  ---	Rank actors based on their average ratings in Indian movies released in India.
 
 with  actr_avg_rating as 
( SELECT 
n.name as actor_name, 
sum(r.total_votes) as total_votes,
count(m.id) as movie_count,
round(sum(r.avg_rating * r.total_votes) /
       sum(r.total_votes),2) 
       as actor_avg_rating 
       from names n 
       inner join role_mapping rm on n.id = rm.name_id
       inner join movies m on m.id =rm.movie_id
       inner join ratings r on r.movie_id = m.id
       where category = 'actor' and  lower(country) like '%india%'
       group by actor_name )
 
 select * ,rank() over ( order by actor_avg_rating desc, total_votes desc)as actor_rank
  from actr_avg_rating
  where movie_count >= 5
 limit 1;
 
 
 Q 5.6----	Identify the top five actresses in Hindi movies released in India based on their average ratings


 with  actr_avg_rating as 
( SELECT 
n.name as actress_name, 
sum(r.total_votes) as total_votes,
count(m.id) as movie_count,
round(sum(r.avg_rating * r.total_votes) /
       sum(r.total_votes),2) 
       as actress_avg_rating 
       from names n 
       inner join role_mapping rm on n.id = rm.name_id
       inner join movies m on m.id =rm.movie_id
       inner join ratings r on r.movie_id = m.id
       where category = 'actress' 
       and lower(languages) like '%hindi%'
       group by actress_name )
 
 select * ,rank() over ( order by actress_avg_rating desc, total_votes desc)as actor_rank
  from actr_avg_rating
  where movie_count >= 3;
  
  
  #########SEGMENT--6 -##########

Q 6.1-- -	Classify thriller movies based on average ratings into different categories.
 
 SELECT title as movie_name, avg_rating as rating,
 case
 when avg_rating > 8 then 'Super_hit'
 when avg_rating  between 7 and 8 then  'hit'
 when avg_rating between 5 and 7 then 'one_time_watch'
 else 'flop'
 end as genre_cetagory
 FROM movies m
 inner join genre g on m.id = g.movie_id
 inner join ratings r on r.movie_id = m.id 
 where lower(genre) like '%thriller%' and total_votes >25000
 order by rating desc;
 

 
 Q 6.2  -	analyse the genre-wise running total and moving average of the average movie duration.
 
 with genre_summary as 
 (select genre, avg(duration) as avg_duration from genre g
 left join movies m on g.movie_id = m.id
 group by genre)
 
 select genre, avg_duration,
 sum(avg_duration) over (order by avg_duration desc ) as running_total,
 avg(avg_duration) over (order by avg_duration desc) as moving_average
 from genre_summary;
 
 
Q 6.3 -	Identify the five highest-grossing movies of each year that belong to the top three genres
 
 with top_genre as 
 (select genre, count(m.id) as movie_count 
 from genre g left join movies m on g.movie_id =m.id
 group by  genre 
 order by movie_count desc limit 3)
 
 select * from 
 ( select genre,year ,m.title as movie_name,
 worlwide_gross_income,
 rank() over ( partition by genre ,year order by 
 cast(replace(trim(worlwide_gross_income),"$"," ")as unsigned)
 desc) as movie_rank from 
 movies m inner join genre g on g.movie_id =m.id
 where g.genre in ( select genre from top_genre)) t 
 where movie_rank <=5;
  

Q 6.4-	Determine the top two production houses that have produced the highest number of hits among multilingual movies
 
 select production_company from
 (
 select production_company, count(m.id) as movie_count,
 row_number() over ( order by count(m.id) desc ) as prd_rnk
 from movies m 
 inner join ratings r on r.movie_id = m.id
 where median_rating >8 and 
 production_company is  not null and
 lower( languages) like '%,%' 
 group by 1 ) t
 where prd_rnk <=2;
 
 
 
 Q 6.5-- -	Identify the top three actresses based on the number of Super Hit movies (average rating > 8) in the drama genre.
 
 with  actr_avg_rating as 
( SELECT 
n.name as actress_name, 
sum(r.total_votes) as total_votes,
count(m.id) as movie_count,
round(sum(r.avg_rating * r.total_votes) /
       sum(r.total_votes),2) 
       as actress_avg_rating 
       from names n 
       inner join role_mapping rm on n.id = rm.name_id
       inner join movies m on m.id =rm.movie_id
       inner join ratings r on r.movie_id = m.id
       inner join genre g on g.movie_id = m.id
       where category = 'actress' and  lower(genre) like '%drama%'
       and avg_rating >8 
       group by actress_name )
       
       select * , row_number () over ( order by actress_avg_rating desc,
       total_votes desc) as actress_rank from actr_avg_rating limit 3;
  

Q 6.6 -	Retrieve details for the top nine directors based on the number of movies, including average inter-movie duration, ratings, and more.

with top_directors as 
( select director_id,director_name
from (select n.id as director_id ,n.name as director_name,
count(m.id)as movie_count,
row_number() over (order by count(m.id)desc ) as director_rank
from names n
inner join director_mapping d on id=d.name_id
inner join movies m on m.id =  d.movie_id
group by 1,2) t
where director_rank <=9),


 movie_summary as 
(select n.id as director_id, n.name as director_name,
m.id as movie_id,
r.avg_rating ,
r.total_votes,
m.duration,
m.date_published,
lead(date_published) over (partition by n.id order by m.date_published) as next_date_published,
datediff(lead(date_published) over (partition by n.id order by m.date_published),
m.date_published) as INTER_MOVIE_DAYS
from  names n
inner join director_mapping d on n.id = d.name_id 
inner join movies m on m.id = d.movie_id 
inner join ratings r on r.movie_id = m.id
where n.id in (select director_id from top_directors) 
)


select 
director_id , 
director_name ,
count(distinct movie_id) as number_of_movies,
avg(inter_movie_days) as avg_inter_movie_days,
round(sum(avg_rating*total_votes)/sum(total_votes),2)
AS directors_avg_rating,
sum(total_votes) as total_votes,
max(avg_rating) as max_rating,
sum(duration)as total_movie_duration from
movie_summary
group by 1,2 
order by number_of_movies desc,
directors_avg_rating desc ;



########SEGMENT --7##########
#### QUESTION -	Based on the analysis, provide recommendations for the types of content Bolly movies should focus on producing.

AS THE PER MY ANALYSIS I CAME ACROSS A.L. VIJAY IS THE TOP DIRECTOR AS PER 
PUBLIC RATING  IF THE BOLLY MOVIES WANTS HITS BY PUBLIC PULSE THEY SHOULD 
AQUIRE KNOWLEDGE ON CONTENT ON  VIJAY MOVIES.Chiris Hemsworth, Robert Downey ,
 CHiris evans are the top three directors in top genres with movie count 12 
 
#1>>
COMINING TO PRODUCTION HOUSES Star cinema and Ave Fenix Pictures 
GAVE MORE HITS CAMPARE TO  THE OTHERS AS THEY ARE TOP PRODUCTION HOUSES 
IN MULTILANGAL SO THAT THEY WILL INTERSSTED IN PRODUCING  MORE
BOLLY MOVIES.

#2>>
IF BOLLY MOVIES COME UP WITH ACTION ,DRAMA, COMEDY GENRE  IT WILL DEFINITELY 
ENTERTAIN AS THEY ARE TOP GENRES PEOPLE ARE MORE INTERESTED IN THESE GENRES 
COMPARATIVELY BEFORE 

 #3>> 
 IN HINDI MOVIES TOP ACTRESSES WHO HITS OR ENTERTAIN PEOPLE BY THEIR PERFORMANCE 
 ARE taapsee pannu , kriti sanon, divya dutta, Sradda Kapoor, Krithi karbanda. 
 WITH ANY OF THOSE ACTRESSES WILL HIT SOON THE INDUSTRY.
  
#4>>> 
IN INDIAN MOVIES VIJAY SEYHUPAYHI RANKED AS NUMBER ONE AND ALSO MOHAN LAL AND MAMMOTTY 
ARE ALSO TOP ACTORS BASED ON THEIR MEDIAN RATINGS . WE CAN CONSIDER THESE ACTORS WILL 
BOX OFFICES IN BOLLY MOVIES
       
#5>>  
FINALLY I CAME INTO MY CONCLUSION COBMINATION OF ABOVE ACTORS AND ACTRESESS AND DIRECTORS 
AND PRODUCTION HOUSES IN TOP GENRES WILL ENTERTAIN MORE 
