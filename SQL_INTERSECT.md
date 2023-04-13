# ВЫБИРАЕМ ОБЩИЕ ДАННЫЕ
Предположим, нам надо вывести совпадающие по названию города и штаты.
```sql
SELECT 
         c.city_name object_name /*выбираем столбец city_name, задаём ему алиас object_name*/
FROM 
         sql.city c /*из схемы sql и таблицы city, задаём таблице алиас с*/

INTERSECT /*оператор присоединения*/

SELECT 
         cc.state /*выбираем столбец state*/
FROM 
         sql.city cc /*из схемы sql и таблицы city, задаём таблице алиас с*/
ORDER BY 1
```
Как видим, с помощью оператора *INTERSECT* мы вывели названия городов и штатов, которые совпадают: **New York**, **Washington** и **Wyoming**. Присмотримся к нему внимательнее.

Синтаксис запроса с оператором *INTERSECT* выглядит следующим образом:
```sql
SELECT 
         n columns
FROM 
         table_1

INTERSECT

SELECT 
         n columns
FROM 
         table_2
```
Вернёмся к нашему примеру с продажами канцтоваров.

С помощью оператора *INTERSECT* мы можем вывести те позиции, которые продавались и в мае, и в июне. Визуализировать это действие можно примерно так:

Оператор *INTERSECT* оставляет только те строки, которые являются общими для двух запросов (в нашем примере это Тетрадь).

Как *EXCEPT*, так и *INTERSECT* убирают дубликаты, если они имеются.

Напишите запрос, который выведет список id городов, в которых есть и клиенты, и доставки, и водители.
```sql
SELECT 
    c.city_id 
FROM 
    sql.city c  
INTERSECT 
SELECT 
    cc.city_id 
FROM 
    sql.customer cc 
INTERSECT
SELECT 
    d.city_id
FROM 
    sql.driver d
```
Выведите zip-код, который есть как в таблице с клиентами, так и в таблице с водителями.
```sql
SELECT 
  c.zip
FROM
  sql.customer c
INTERSECT
SELECT
  d.zip_code
FROM
  sql.driver d
```
Запишем структуру запроса с учётом полученных знаний.
```sql
SELECT 
         N columns
FROM 
         table_1

UNION / UNION ALL / EXCEPT / INTERSECT 

SELECT 
         N columns
FROM 
         table_2
```
