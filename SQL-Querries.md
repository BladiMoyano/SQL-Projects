# SQL Querries

In this project, I practiced my SQL skills by working with real-world datasets and writing basic queries. The datasets cover 120 years of Olympic history and offer plenty of interesting details to explore.

**Dataset 1:** athlete_events.csv – Contains information about athletes and the medals they won.

**Dataset 2:** noc_regions.csv – Includes details about country names and regions.

** one picture of each data set ** 

## List of all these 20 queries mentioned below:

- **1. How many Olympic Games have been held?**  
- **2. List all Olympic Games held so far.**  
- **3. How many nations participated in each Olympic Games?**  
- **4. Identify the years with the highest and lowest number of participating countries.**  
- **5. Which nation has participated in all Olympic Games?**  
- **6. Which sport has been played in all Summer Olympics?**  
- **7. Which sports were played only once in the Olympics?**  
- **8. Total number of sports played in each Olympic Games.**  
- **9. Details of the oldest athlete to win a gold medal.**  
- **10. Ratio of male to female athletes in all Olympic Games.**  
- **11. Top 5 athletes with the most gold medals.**  
- **12. Top 5 athletes with the most total medals (gold/silver/bronze).**  
- **13. Top 5 most successful countries based on total medals won.**  
- **14. Total gold, silver, and bronze medals won by each country.**  
- **15. Total gold, silver, and bronze medals won by each country in each Olympic Games.**  
- **16. Identify the countries that won the most gold, silver, and bronze medals in each Olympic Games.**  
- **17. Identify the countries that won the most gold, silver, bronze, and total medals in each Olympic Games.**  
- **18. Countries that have never won a gold medal but have won silver or bronze.**  
- **19. Sport/event where India has won the highest number of medals.**  
- **20. Breakdown of Olympic Games where India won medals in hockey and how many in each game.**  

## 1. How many Olympic Games have been held?
```sql
SELECT count(distinct games) as total_games
FROM olympic_history;
```
* Output:

## 2. List all Olympic Games held so far.
```sql
SELECT distinct `year`, season, city
FROM olympic_history
order by `year`;
```
* Output:

## 3. Mention the total no of nations who participated in each olympics game?.
```sql
with all_countries as
        (select games, nr.region
        from olympics_history oh
        join olympics_history_noc_regions nr ON nr.noc = oh.noc
        group by games, nr.region)
    select games, count(1) as total_countries
    from all_countries
    group by games
    order by games;
```
* Output:














