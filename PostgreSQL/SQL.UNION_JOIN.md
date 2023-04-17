# Сложные объединения таблиц

В данном блоке мы будем работать с данными о компании, организующей перевозки грузов. Интересующие нас данные хранятся в таблицах *city*, *customer*, *driver*, *shipment*, *truck*. Давайте внимательно их изучим.

### Таблица **city** — это справочник городов. Структура справочника представлена ниже.

* __city_id__ - _integer_ - 	уникальный идентификатор города, первичный ключ
* __city_name__ - _text_ -	название города
* __state__ - _text_ - штат, к которому относится город
* __population__ - _integer_ - население города
* __area__ - _numeric_ - площадь города

### Таблица **customer** — это справочник клиентов. У компании, с данными которой мы работаем, только корпоративные клиенты, поэтому в таблице нет привычных данных о возрасте и поле. Справочник содержит следующие поля:

* __cust_id__ - _integer_ - 	уникальный идентификатор клиента, первичный ключ
* __cust_name__ - _text_ - 	название клиента
* __annual_revenue__ - _numeric_ - 	ежегодная выручка
* __cust_type__ - _text_ - тип пользователя
* __address__ - _text_ - адрес
* __zip__ - _integer_ - 	почтовый индекс
* __phone__ - _text_ - 	телефон
* __city_id__ - _integer_ - 	идентификатор города, внешний ключ к таблице *city*

### Следующая таблица — *driver* — справочник водителей. Перечень сведений, содержащихся в таблице, представлен ниже.
* __driver_id__ - _integer_ - уникальный идентификатор водителя, первичный ключ
* __first_name__ - _text_ - имя водителя
* __last_name__ - _text_ - фамилия водителя
* __address__ - _text_ - адрес водителя
* __zip_code__ - _integer_ - почтовый индекс водителя
* __phone__ - _text_ - телефон водителя
* __city_id__ - _integer_ - идентификатор города водителя, внешний ключ к таблице 
### В таблице *truck* хранится информация о грузовиках, на которых осуществляются перевозки. Данные о них представлены в следующем виде:
* __truck_id__ - _integer_ - Уникальный идентификатор грузовика, первичный ключ
* __make__ - _text_ - Производитель грузовика
* __model_year__ - _integer_ - Дата выпуска грузовика
### Tаблица в датасете, *shipment*, — таблица с данными непосредственно о доставках. Она описывает взаимодействие всех перечисленных сущностей, а потому содержит наибольшее количество ссылок на другие таблицы.
* __ship_id__ - _integer_ - уникальный идентификатор доставки, первичный ключ
* __cust_id__ - _integer_ - идентификатор клиента, которому отправлена доставка, внешний ключ к таблице customer
* __weight__ - _numeric_ - вес посылки
* __truck_id__ - _integer_ - идентификатор грузовика, на котором отправлена доставка, внешний ключ
к таблице truck
* __driver_id__ - _integer_ - идентификатор водителя, который осуществлял доставку, внешний ключ к таблице driver
* __city_id__ - _integer_ - идентификатор города в который совершена доставка, внешний ключ
к таблице city
* __ship_date__ - _date_ - дата доставки
 
Укажите название города с максимальным весом единичной доставки.
```sql
SELECT 
    c.city_name,
    MAX(s.weight)
FROM sql.shipment s
    JOIN sql.city c ON c.city_id = s.city_id
GROUP BY c.city_name
ORDER BY MAX DESC
LIMIT 1
```
Сколько различных производителей грузовиков перечислено в таблице truck?
```sql
SELECT 
    count(DISTINCT make)
FROM sql.truck
```
Как зовут водителя (first_name), который совершил наибольшее количество доставок одному клиенту?
```sql
SELECT 
    d.first_name,
    COUNT(s.driver_id),
    COUNT(s.cust_id)
FROM
    sql.driver d
    JOIN sql.shipment s ON s.driver_id = d.driver_id
GROUP BY d.first_name, s.driver_id, s.cust_id
ORDER BY 2 DESC, 3 DESC
```
Укажите даты первой и последней по времени доставок в таблице shipment.
```sql
SELECT
    s1.ship_date,
    s2.ship_date
FROM 
    sql.shipment s1 
    CROSS JOIN sql.shipment s2 
ORDER BY 1, 2 DESC
LIMIT 1
```
Укажите имя клиента, получившего наибольшее количество доставок за 2017 год.
```sql
SELECT 
    c.cust_name,
    COUNT(*)
FROM 
    sql.shipment s
    JOIN sql.customer c ON c.cust_id = s.cust_id 
WHERE s.ship_date BETWEEN '01-01-2017' AND '12-31-2017'
GROUP BY c.cust_name
ORDER BY 2 DESC
```
# ПРИНЦИП И УСЛОВИЯ РАБОТЫ UNION
Допустим, мы хотим собрать из справочников по книгам и фильмам один, так чтобы в нём содержались названия произведений, а также их описание — книга или фильм.

Для этого напишем простой запрос:

```sql
SELECT book_name object_name, 'книга' object_descritption /*выбираем столбец с названием book_name, задаём алиас для столбца object_name, задаём во второй колонке объект ‘книга’ с алиасом для столбца object_descritption*/
FROM public.books /*из схемы public и таблицы books*/
UNION ALL /*оператор присоединения*/
SELECT movie_title, 'фильм' /*выбираем столбец movie_title, сами задаём во второй колонке объект ‘фильм’*/
FROM sql.kinopoisk /*из схемы sql и таблицы kinopoisk*/
```
Визуально произведённое нами действие можно представить следующим образом:

В запросе мы использовали оператор *UNION ALL* — он присоединяет любой результат запроса к другому «снизу» при условии, что у них *одинаковая структура*, а именно:
* одинаковый тип данных:
* одинаковое количество столбцов:
* одинаковый порядок столбцов согласно типу данных:
## ВИДЫ UNION
Оператор присоединения существует в двух вариантах:

* *UNION* выводит только **уникальные** записи;
* *UNION ALL* присоединяет **все строки** последующих таблиц к предыдущим, без ограничений по уникальности.

**Важно!** *UNION* оставляет только уникальные значения, а потому требует дополнительных вычислительных мощностей и памяти (в данном случае можно провести аналогию с *DISTINCT*). Поэтому если вы уверены в отсутствии дубликатов в данных или они вам не важны, предпочтительнее использовать *UNION ALL*.
## СИНТАКСИС
```sql
SELECT
         n columns
FROM 
         table_1

UNION ALL

SELECT 
         n columns
FROM 
         table_2
...

UNION ALL

SELECT 
         n columns
FROM 
         table_n
```
Результатом выполнения такого запроса будут строки *table_1, table_2, ..., table_n*, соединённые одни под другими и выведенные в единой выдаче.

**Важно!** Названия итоговых колонок в выводе будут такие же, как в первом блоке *SELECT*, даже если они отличаются в других блоках подзапросов.
```sql
SELECT
         c.city_id object_name,  'id города' object_type /*выбираем колонку city_id и задаём ей алиас object_name, сами задаём объект 'id города' и название столбца object_type*/
FROM 
         sql.city c /*из схемы sql и таблицы city, задаём алиас таблице — с*/

UNION ALL /*оператор присоединения*/

SELECT
         d.driver_id other_name,  'id водителя' other_type /*выбираем колонку driver_id и задаём ей алиас other_name, сами задаём объект 'id водителя' и название столбца other_type*/
FROM 
         sql.driver d  /*из схемы sql и таблицы driver, задаём алиас таблице — d*/

UNION ALL /*оператор присоединения*/

SELECT
         s.ship_id,  'id доставки' /*выбираем колонку ship_id, сами задаём объект 'id доставки'*/
FROM 
         sql.shipment s /*из схемы sql и таблицы shipment, задаём алиас таблице — s*/

UNION ALL /*оператор присоединения*/

SELECT
         c.cust_id,  'id клиента' /*выбираем колонку cust_id, сами задаём объект 'id клиента'*/
FROM 
         sql.customer c /*из схемы sql и таблицы customer, задаём алиас таблице — c*/

UNION ALL /*оператор присоединения*/

SELECT
         t.truck_id,  'id грузовика' /*выбираем колонку truck_id, сами задаём объект 'id грузовика'*/
FROM 
         sql.truck t /*из схемы sql и таблицы truck, задаём алиас таблице — t*/
ORDER BY 1 /*сортировка по первому столбцу*/
```
**Обратите внимание!** Несмотря на исходные названия колонок other_name и other_type во втором подзапросе, в выводе мы получим названия, которые дали в первом блоке: object_name и object_type.

Другая особенность — в применении сортировки ORDER BY: она всегда будет относиться к итоговому результату всего запроса с UNION ALL.
```sql
SELECT book_name object_name, 'книга' objetc_description
FROM public.books 
UNION ALL 
SELECT movie_title, 'фильм' 
FROM sql.kinopoisk 
ORDER BY 1 
LIMIT 1
```
Всё бы хорошо, только в таком случае отсортирован будет весь общий справочник, а в выводе останется одна строка с названием объекта, идущим первым по алфавиту.

А если мы не хотим общую сортировку? Может, нам нужны строки с названием как фильма, так и книги, идущих первыми по алфавиту.

Нет ничего проще — отсортируем каждую часть запроса по отдельности и объединим результаты! Просто добавим ORDER BY и LIMIT ещё и в первую часть запроса:
```sql
SELECT book_name object_name, 'книга' objetc_description
FROM public.books 
ORDER BY 1 
LIMIT 1
UNION ALL 
SELECT movie_title, 'фильм' 
FROM sql.kinopoisk 
ORDER BY 1 
LIMIT 1
```
Вместо результата получим сообщение о синтаксической ошибке: **"...syntax error at or near "UNION"..."** Очевидно, этот фокус не удался.

Проблему можно решить одним (ну, почти) движением руки — просто добавив скобки вокруг каждой из частей запроса.
```sql
(SELECT book_name object_name, 'книга' objetc_description
FROM public.books 
ORDER BY 1 
LIMIT 1)
UNION ALL 
(SELECT movie_title, 'фильм' 
FROM sql.kinopoisk 
ORDER BY 1 
LIMIT 1)
```
Всё получилось!

Напишите запрос, который создает уникальный алфавитный справочник всех городов, штатов, имён водителей и производителей грузовиков. Результатом запроса должны быть два столбца: название и тип объекта (city, state, driver, truck). Отсортируйте список по названию объекта, а затем — по типу.
```sql
(SELECT c.city_name object_name, 'city' objetc_type
FROM sql.city c 
ORDER BY 1, 2)
UNION 
(SELECT c.state, 'state' 
FROM sql.city c 
ORDER BY 1, 2)
UNION
(SELECT d.first_name, 'driver' 
FROM sql.driver d
ORDER BY 1, 2)
UNION
(SELECT t.make, 'truck' 
FROM sql.truck t
ORDER BY 1, 2)
```
Напишите запрос, который соберёт имена всех упомянутых городов и штатов из таблицы city. Результатом запроса должен быть один столбец object_name, отсортированный в алфавитном порядке.
```sql
SELECT city_name object_name
FROM sql.city  
UNION ALL 
SELECT state
FROM sql.city 
ORDER BY 1
```
Выполнив предыдущий запрос, мы получили города с одинаковыми названиями, но находящиеся в разных штатах, а также большое количество дублирующихся названий штатов. Перепишите предыдущий запрос так, чтобы остались только уникальные названия городов и штатов. Результатом запроса должен быть один столбец object_name, отсортированный в алфавитном порядке.
```sql
(SELECT city_name object_name
FROM sql.city
ORDER BY 1)  
UNION 
(SELECT state
FROM sql.city 
ORDER BY 1)
```
## ПОЧЕМУ ТАК ВАЖЕН ТИП ДАННЫХ?
Как мы уже знаем, **UNION** может быть использован только в случае полного соответствия столбцов и их типов в объединяемых запросах. Hапишем запрос, который позволит получить нужный нам результат.
```sql
SELECT 
         c.city_id /*выбираем столбец city_id*/
FROM
         sql.city c /*из схемы sql  и таблицы city, задаём таблице алиас с*/ 

UNION ALL /*оператор присоединения*/

SELECT 
         cc.city_name /*выбираем столбец city_name*/
FROM
         sql.city cc /*из схемы sql и таблицы city, задаём таблице алиас сс*/
```
Вместо результата вы получите сообщение об ошибке: **"ERROR: UNION types integer and text cannot be matched"**. Дело в том, что мы попытались объединить числовой и строковый типы в одной колонке, а это невозможно.

**Важно!** Любой тип данных может быть приведён к текстовому формату — эту возможность целесообразно использовать для соединения разнородных сущностей. Главное — помнить, что сортировка текста **отличается** от сортировки чисел и дат.

Немного подправим запрос, чтобы получить желаемый результат.
```sql
SELECT 
         c.city_id::text /*выбираем столбец city_id, переводим city_id из числового в текстовый формат*/
FROM
         sql.city c /*из схемы sql  и таблицы city, задаём таблице алиас с*/

UNION ALL /*оператор присоединения*/

SELECT 
         cc.city_name /*выбираем столбец city_name*/
FROM
         sql.city cc /*из схемы sql и таблицы city, задаём таблице алиас сс*/
```
Напишите запрос, который объединит в себе все почтовые индексы водителей и их телефоны в единый столбец-справочник contact. Также добавьте столбец с именем водителя first_name и столбец contact_type с типом контакта (phone или zip в зависимости от типа). Отсортируйте список по столбцу с контактными данными в порядке возрастания, а затем — по имени водителя.
```sql
SELECT zip_code::text contact, first_name, 'zip' contact_type
FROM sql.driver  
UNION ALL
SELECT phone contact, first_name, 'phone' contact_type
FROM sql.driver 
```
## ВОЗМОЖНОСТИ UNION
Помимо соединения разнородных сущностей в единый справочник, *UNION ALL* часто используется для **подведения промежуточных итогов и выведения результатов агрегатных функций**.

Попробуем вывести обобщённые данные о населении по всем городам, с детализацией до конкретного города.
```sql
SELECT
         c.city_name,
         c.population /*выбираем столбцы city_name, population*/
FROM
         sql.city c /*из схемы sql и таблицы city, задаём таблице алиас с*/

UNION ALL /*оператор присоединения*/

SELECT
         'total',
         SUM(c.population) /*сами задаём объект ‘total’, суммируем все значения столбца population*/
FROM
         sql.city c /*из схемы sql и таблицы city, задаём таблице алиас с*/
ORDER BY 2 DESC /*сортируем по второму столбцу в убывающем порядке (чтобы итоговая сумма была в начале)*/
```
Напишите запрос, который выводит общее число доставок total_shipments, а также количество доставок в каждый день. Необходимые столбцы: date_period, cnt_shipment. Не забывайте о единой типизации. Упорядочите по убыванию столбца date_period.
```sql
SELECT 
    s.ship_date::text date_period,
    COUNT(*) cnt_shipment
FROM 
    sql.shipment s 
GROUP BY 1
UNION ALL 
SELECT 
    'total_shipments',
    COUNT(*) 
FROM 
    sql.shipment s 
ORDER BY 1 DESC 
```
Например, с помощью UNION можно отобразить, у кого из водителей заполнен столбец с номером телефона.
```sql
SELECT
         d.first_name,
         d.last_name,
         'телефон заполнен' phone_info /*выбираем столбцы first_name, last_name, сами выводим объект ‘телефон заполнен’*/
FROM
         sql.driver d /*из схемы sql и таблицы driver, задаём алиас d*/
WHERE d.phone IS NOT NULL /*условие, что телефон заполнен*/

UNION /*оператор присоединения (уникальные значения)*/

SELECT
         d.first_name,
         d.last_name,
         'телефон не заполнен' phone_info /*выбираем столбцы first_name, last_name, сами выводим объект ‘телефон не заполнен’*/
FROM
         sql.driver d /*из схемы sql и таблицы driver, задаём алиас d*/
WHERE d.phone IS NULL /*условие, что телефон не заполнен*/
```
Напишите запрос, который выведет все города и штаты, в которых они расположены, а также информацию о том, была ли осуществлена доставка в этот город:

*  если в город была осуществлена доставка, то выводим 'доставка осуществлялась';
*  если нет — выводим 'доставка не осуществлялась'.

Столбцы к выводу: city_name, state, shipping_status. Отсортируйте в алфавитном порядке по городу, а затем — по штату.
```sql
SELECT 
    c.city_name AS city_name, 
    c.state AS state, 
    'доставка осуществлялась' AS shipping_status
FROM 
    sql.city c
    LEFT JOIN sql.shipment s ON c.city_id = s.city_id
WHERE s.city_id IS NOT NULL
UNION
SELECT 
    c.city_name AS city_name, 
    c.state AS state, 
    'доставка не осуществлялась' AS shipping_status
FROM 
    sql.city c
    LEFT JOIN sql.shipment s ON c.city_id = s.city_id
WHERE s.city_id IS NULL
ORDER BY 1, 2
```
Напишите запрос, который выводит два столбца: city_name и shippings_fake. Выведите города, куда совершались доставки. Пусть первый столбец содержит название города, а второй формируется так:

* если в городе было более десяти доставок, вывести количество доставок в этот город как есть;
* иначе — вывести количество доставок, увеличенное на пять.

Отсортируйте по убыванию получившегося «нечестного» количества доставок, а затем — по имени в алфавитном порядке.
```sql
SELECT 
    c.city_name AS city_name, 
    COUNT(s.ship_id) shippings_fake
FROM 
    sql.city c
    JOIN sql.shipment s ON c.city_id = s.city_id
GROUP BY 
  c.city_name
HAVING COUNT(s.ship_id) > 10
UNION
SELECT 
    c.city_name AS city_name, 
    COUNT(s.ship_id)+5 shippings_fake
FROM 
    sql.city c
    JOIN sql.shipment s ON c.city_id = s.city_id
GROUP BY c.city_name 
HAVING COUNT(s.ship_id) <= 10
ORDER BY 
  shippings_fake DESC,
  city_name ASC
```