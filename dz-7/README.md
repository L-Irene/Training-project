### Домашнее задание ###
Посчитать кол-во очков по всем игрокам за текущий год и за предыдущий.      
#### Цель: ####   
Использовать функцию LAG и CTE.    

#### Описание/Пошаговая инструкция выполнения домашнего задания: ####   
1) cоздайте таблицу и наполните ее данными     
<...>;    
2) заполнить данными    
<...>;   
3) написать запрос суммы очков с группировкой и сортировкой по годам;      
4) написать cte показывающее тоже самое;     
5) используя функцию LAG вывести кол-во очков по всем игрокам за текущий год и за предыдущий.     

```sql
CREATE TABLE statistic(
    player_name VARCHAR(100) NOT NULL,
    player_id INT NOT NULL,
    year_game SMALLINT NOT NULL CHECK (year_game > 0),
    points DECIMAL(12,2) CHECK (points >= 0),
    PRIMARY KEY (player_name, year_game)
);
```    
```sql    
INSERT INTO statistic(player_name, player_id, year_game, points)   
VALUES
    ('Mike',1,2018,18),
    ('Jack',2,2018,14),
    ('Jackie',3,2018,30),
    ('Jet',4,2018,30),
    ('Luke',1,2019,16),
    ('Mike',2,2019,14),
    ('Jack',3,2019,15),
    ('Jackie',4,2019,28),
    ('Jet',5,2019,25),
    ('Luke',1,2020,19),
    ('Mike',2,2020,17),
    ('Jack',3,2020,18),
    ('Jackie',4,2020,29),
    ('Jet',5,2020,27);
```    
```sql     
SELECT year_game, SUM(points) AS total_points
FROM statistic
GROUP BY year_game
ORDER BY year_game;
```
```sql      
WITH year_points AS (
    SELECT year_game, SUM(points) AS total_points
    FROM statistic
    GROUP BY year_game
)
SELECT year_game, total_points
FROM year_points
ORDER BY year_game;
```
```sql 
SELECT player_name, year_game, points,
    LAG(points) OVER (PARTITION BY player_name ORDER BY year_game) AS prev_points
FROM statistic
ORDER BY player_name, year_game;
```
