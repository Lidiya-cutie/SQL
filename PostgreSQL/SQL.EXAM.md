Допустим, у нас есть две таблицы с одинаковыми полями (хранят один и тот же вид информации): A {name, description} и B {name, description}.

Какой запрос позволит вывести все поля (отсортированные по первому полю) из этих двух объединённых таблиц без дубликатов?

```sql
SELECT *
FROM A
UNION
SELECT *
FROM B
ORDER BY name
```

Выберите верное утверждение для запроса:
```sql
select *
from t1 F1
```
Выводятся все столбцы из таблицы t1, которой присвоен алиас F1

Что такое JOIN?

Операция соединения

Выберите неверное описание типа JOIN:

FULL JOIN возвращает только те записи, для которых не нашлось пар.

Что такое агрегатные функции?

Функции, которые работают с набором данных, превращая их в одно итоговое значение.

Продолжите фразу: «К агрегатным функциям относятся...»

MIN, MAX, COUNT, AVG, SUM.

Что выведет следующий запрос?
```sql
SELECT cust_id, count(*) cnt
FROM Sales 
WHERE date > '2022-01-01'
HAVING count(*) > 1 
GROUP BY cust_id
```
Ничего, запрос составлен неверно — HAVING указывается после группировки.


Hапишите запрос, который покажет информацию по трём самым старым фильмам, у которых есть рейтинг. Выведите все столбцы таблицы movies.

Подсказка: Все столбцы лучше всего вывести через select *.
```sql
select *
from sqlprotest.movies
where rating is not null
order by year
limit 3
```

Напишите запрос, выводящий для каждого жанра средний рейтинг и количество фильмов этого жанра.

Столбцы к выводу:

genre_name (название жанра),
average_rating (средний рейтинг),
movie_count (количество фильмов).
Результат отсортируйте по убыванию среднего рейтинга.
```sql
select g.name genre_name,
avg(rating) average_rating,
count(m.id) movie_count 
from sqlprotest.genres g
  left join sqlprotest.movie_genres mg on g.id = mg.genre_id
  left join sqlprotest.movies m on mg.movie_id = m.id
group by 1
order by 2 desc
```

Напишите запрос, чтобы вывести все названия фильмов и их рейтинги. Если у фильма нет рейтинга, то проставьте 0 в качестве значения рейтинга.

Результат отсортируйте по названию фильма в алфавитном порядке.

Примечание: Буква "ё" в SQL не является алфавитной и не сортируется, не обращайте внимание на это во время решения задания.
```sql
select m.name,
coalesce(m.rating,0)
from sqlprotest.movies m
order by m.name
```

Напишите запрос, который выведет количество фильмов в каждом жанре для случаев, когда в жанре представлено три или больше фильмов.

Столбцы к выводу:

genre_name (имя жанра)
movies_count (количество фильмов).
Результат отсортируйте по убыванию количества фильмов.
```sql
select g.name genre_name,
count(m.id) movie_count 
from sqlprotest.genres g
  left join sqlprotest.movie_genres mg on g.id = mg.genre_id
  left join sqlprotest.movies m on mg.movie_id = m.id
group by 1
having count(m.id) >= 3
order by 2 desc
```
Напишите запрос, с помощью которого можно выбрать фильмы, не относящиеся к жанру 'Криминал'. Выведите все столбцы таблицы movies.

Результат должен быть отсортирован по названию фильмов в алфавитном порядке.

Подсказка: Все столбцы лучше всего вывести через *.
```sql
select movies.*
from sqlprotest.movies
except
select movies.*
from sqlprotest.genres
  join sqlprotest.movie_genres on genres.id = movie_genres.genre_id
  join sqlprotest.movies on movie_genres.movie_id = movies.id
where genres.name like 'Криминал'
```