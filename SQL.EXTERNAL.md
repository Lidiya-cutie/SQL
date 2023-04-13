Выведите города с максимальным и минимальным весом единичной доставки. Столбцы к выводу — city_name, weight.
```sql
(SELECT 
  c.city_name,
  s.weight
FROM
  sql.city c
  JOIN sql.shipment s ON c.city_id = s.city_id
ORDER BY 2 DESC
LIMIT 1)
UNION ALL
(SELECT
  c.city_name,
  s.weight
FROM 
  sql.city c 
  JOIN sql.shipment s ON c.city_id = s.city_id
ORDER BY 2 ASC
LIMIT 1)
```
Выведите идентификационные номера клиентов (cust_id), которые совпадают с идентификационными номерами доставок (ship_id). Столбец к выводу — mutual_id. Отсортируйте по возрастанию.
```sql
SELECT
  c.cust_id mutual_id
FROM
  sql.customer c
INTERSECT
SELECT
  s.ship_id
FROM 
  sql.shipment s
ORDER BY 1
```
Создайте справочник, содержащий уникальные имена клиентов, которые являются производителями (cust_type='manufacturer'), и производителей грузовиков, а также описание объекта — 'КЛИЕНТ' или 'ГРУЗОВИК'. Столбцы к выводу — object_name, object_description. Отсортируйте по названию в алфавитном порядке.
```sql
SELECT 
  cust_name object_name, 
  'КЛИЕНТ' object_description
FROM
  sql.customer 
WHERE cust_type = 'manufacturer'
UNION
SELECT
  make, 'ГРУЗОВИК'
FROM 
  sql.truck
ORDER BY 1
```
Составьте список книжных новинок. Новинками считаются все книги за последние пять лет. Необходимые данные:

* название книги;
* год издания;
* автор;
* жанр.

Вывод отсортируйте по названиям книг. Примечание. При решении задачи текущим следует считать 2021 год.
```sql
SELECT 
    book_name,
    publishing_year,
    author,
    genre
FROM 
    other.books
WHERE publishing_year between 2016 and 2021
ORDER BY book_name
```
или
```sql
select
  b.book_name, 
  b.publishing_year, 
  b.author, 
  b.genre 
from 
  other.books b 
where 
  b.publishing_year >= 2021 - 5 
order by 1
```
Отфильтруйте запрос из предыдущего задания так, чтобы остались только те книги, у которых есть название.
```sql
SELECT 
    book_name,
    publishing_year,
    author,
    genre
FROM 
    other.books
WHERE (publishing_year between 2016 and 2021) AND book_name IS NOT NULL 
ORDER BY book_name
```
или
```sql
select
b.book_name, 
b.publishing_year, 
b.author, 
b.genre 
from 
other.books b 
where 
b.publishing_year >= 2021 - 5 
and b.book_name is not null 
order by 
1
```
Теперь нам надо как-то урезать количество оставшихся книг.

Может, по рейтингу автора? Неплохо, только мы не знаем, какие категории авторов у нас есть. Давайте выясним это.

Выберите значения рейтинга авторов, имеющиеся в нашей базе. Отсортируйте вывод по алфавиту.
```sql
SELECT 
    DISTINCT author_rating
FROM 
    other.books
WHERE (publishing_year between 2016 and 2021) AND book_name IS NOT NULL 
ORDER BY author_rating
```
Возьмём для рекламного буклета только книги отличных авторов! Оставьте в выборке новых книг только авторов с рейтингом 'Excellent'.
```sql
SELECT 
    book_name,
    publishing_year,
    author,
    genre
FROM 
    other.books
WHERE (publishing_year between 2016 and 2021) 
    AND book_name IS NOT NULL 
    AND author_rating = 'Excellent'
ORDER BY book_name
```
Добавьте в имеющуюся выборку известных авторов (со значением рейтинга 'Famous').
```sql
SELECT 
    book_name,
    publishing_year,
    author,
    genre
FROM 
    other.books
WHERE (publishing_year between 2016 and 2021) 
    AND book_name IS NOT NULL 
    AND (author_rating = 'Excellent' OR author_rating = 'Famous')
ORDER BY book_name
```
или
```sql
select
b.book_name, 
b.publishing_year, 
b.author, 
b.genre 
from 
other.books b 
where 
b.publishing_year >= 2021 - 5 
and b.book_name is not null 
and author_rating IN ('Excellent', 'Famous') 
order by 
1
```
Определите, сколько книг из выборки для рекламы попадает в каждую категорию рейтинга автора. Нам понадобятся следующие данные:

рейтинг автора (author_rating);
количество книг (cnt).
Сортировка, как всегда, по алфавиту. Примечание. Пока можете просто закомментировать с помощью -- лишний фильтр по рейтингам и лишние колонки.
```sql
SELECT
  author_rating,
  COUNT(book_id) 
FROM
  other.books
WHERE (publishing_year between 2016 and 2021) 
    AND book_name IS NOT NULL 
    --AND (author_rating = 'Excellent' OR author_rating = 'Famous')
GROUP BY 1
ORDER BY author_rating
```
Выбираем книги с рейтингом автора отличный (Excellent), известный (Famous) и новый (Novice). И в конце добавим строку об общем количестве книг. В выборке нас по-прежнему интересуют

* название книги,
* год издания,
* автор,
* жанр.
```sql
(SELECT 
    book_name,
    publishing_year,
    author,
    genre
FROM 
    other.books
WHERE (publishing_year between 2016 and 2021) 
    AND book_name IS NOT NULL 
    AND author_rating IN ('Excellent', 'Famous', 'Novice')
ORDER BY book_name)
UNION ALL
(SELECT
  'Total',
  '6',
  null,
  null)
```
Для начала выберите всю информацию о заказах книг, выпущенных не более 10 лет назад. Отсортируйте заказы по дате в обратном порядке.
```sql
SELECT *
FROM
  other.book_orders o
  JOIN other.books b ON o.book_id = b.book_id
WHERE 
   b.publishing_year >= 2021 - 10
ORDER BY o.order_date DESC
```
Теперь оставим в выборке только заказы от 2019 года и позднее.
```sql
SELECT *
FROM
  other.book_orders o
  JOIN other.books b ON o.book_id = b.book_id
WHERE 
    b.publishing_year >= 2021 - 10 AND
    DATE(o.order_date) BETWEEN '2019-01-01' AND '2021-01-01'
ORDER BY o.order_date DESC
```
или
```sql
SELECT *
FROM
    other.book_orders o
    JOIN other.books b ON o.book_id = b.book_id 
WHERE 
    b.publishing_year >= 10
    AND o.order_date  >= TO_DATE('2019-01-01', 'YYYY-MM-DD')
ORDER BY order_date DESC
```
Чтобы понять, заказы за какой период у нас есть, определите дату последнего заказа.
```SQL
SELECT 
    MAX(o.order_date)
FROM
    other.book_orders o
    JOIN other.books b ON o.book_id = b.book_id
WHERE 
    b.publishing_year >= 2021 - 10 AND
    DATE(o.order_date) BETWEEN '2019-01-01' AND '2021-01-01'
```
Посчитайте общее количество заказов за каждый месяц (month). Отсортируйте вывод по месяцам в обратном порядке.
```sql
SELECT 
    EXTRACT(MONTH FROM o.order_date) order_month,
    COUNT(o.order_id) cnt
FROM
  other.book_orders o
  JOIN other.books b ON o.book_id = b.book_id
WHERE 
    b.publishing_year >= 2021 - 10 AND
    DATE(o.order_date) BETWEEN '2019-01-01' AND '2021-01-01'
GROUP BY EXTRACT(MONTH FROM order_date)
ORDER BY order_month DESC
```
Добавьте в предыдущий запрос подсчёт количества разных книг (cnt_dist), заказанных в каждом месяце.
```sql
SELECT 
    EXTRACT(MONTH FROM o.order_date) order_month,
    COUNT(o.order_id) cnt,
    COUNT(DISTINCT o.book_id) cnt_dist
FROM
  other.book_orders o
  JOIN other.books b ON o.book_id = b.book_id
WHERE 
    b.publishing_year >= 2021 - 10 AND
    DATE(o.order_date) BETWEEN '2019-01-01' AND '2021-01-01'
GROUP BY EXTRACT(MONTH FROM order_date)
ORDER BY order_month DESC
```
или 
```sql
select
extract(
MONTH 
from 
order_date
) order_month, 
count(*) cnt, 
count(distinct o.book_id) cnt_dist 
from 
other."book_orders" o 
join other.books b on o.book_id = b.book_id 
where 
b.publishing_year >= 2021 - 10 
and order_date >= to_date('2019-01-01', 'YYYY-MM-DD') 
group by 
extract(
MONTH 
from 
order_date
) 
order by 
order_month desc
```
На основе предыдущего запроса создайте новый, чтобы вычислить, сколько раз заказывали каждую книгу в этом месяце. Столбцы к выводу — order_month, book_name, cnt.
```sql
SELECT 
    EXTRACT(MONTH FROM o.order_date) order_month,
    o.book_name,
    COUNT(o.book_id) cnt
FROM
  other.book_orders o
  JOIN other.books b ON o.book_id = b.book_id
WHERE 
    b.publishing_year >= 2021 - 10 AND
    DATE(o.order_date) BETWEEN '2019-01-01' AND '2021-01-01'
GROUP BY EXTRACT(MONTH FROM order_date), o.book_name
ORDER BY order_month DESC
```
или
```sql
select
extract(
MONTH 
from 
order_date
) order_month, 
b.book_name, 
count(*) cnt 
from 
other."book_orders" o 
join other.books b on o.book_id = b.book_id 
where 
b.publishing_year >= 2021 - 10 
and order_date >= to_date('2019-01-01', 'YYYY-MM-DD') 
group by 
extract(
MONTH 
from 
order_date
), 
b.book_name 
order by 
order_month desc
```
Выберите топ-5 книг по заказам в каждом месяце. Столбцы к выводу — order_month, book_name, cnt, rnk. Отсортируйте вывод по месяцу в обратном порядке и по рангу.
```sql

```
