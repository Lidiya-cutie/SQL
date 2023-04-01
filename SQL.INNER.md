# ОПЕРАТОРЫ
### **INNER JOIN** — это тот же JOIN (слово *inner* в операторе можно опустить).
Для **INNER JOIN** работает следующее **правило**: присоединяются только те строки таблиц, которые удовлетворяют условию соединения. Если в любой из соединяемых таблиц находятся такие строки, которые не удовлетворяют заявленному условию, — они отбрасываются.

В этом модуле мы будем работать с таблицами о футбольных матчах и командах.

Таблицы этого модуля лежат в схеме sql или csv в папке data. Нам понадобятся таблицы teams и matches.

### Таблица **teams** с данными о командах

* __id__ - id команды
* __api_id__ - ключ на таблицу matches
* __long_name__ - полное название команды
* __short_name__ - сокращённое название команды

### Таблица **matches** с данными о матчах

* __id__ - id матча
* __season__ - сезон
* __date__ - дата матча
* __home_team_api_id__ - api_id домашней команды, ключ на таблицу teams
* __away_team_api_id__ - api_id гостевой команды, ключ на таблицу teams
* __home_team_goals__ - количество голов домашней команды
* __away_team_goals__ - количество голов гостевой команды

В таблице *teams* есть данные о 299 различных командах — можем проверить это с помощью запроса.
```SQL
SELECT 
COUNT(DISTINCT id) /*оператор подсчёта строк; оператор исключения повторяющихся строк; столбец id*/
FROM sql.teams /*таблица teams*/
```
ИЛИ
```SQL
SELECT 
COUNT(DISTINCT api_id) /*оператор подсчёта строк; оператор исключения повторяющихся строк; столбец api_id*/
FROM sql.teams
```
Теперь добавим к *teams* таблицу с матчами.

```SQL
SELECT 
    COUNT(DISTINCT t.id) /*оператор подсчёта строк; оператор исключения повторяющихся строк; столбец id*/
FROM 
    sql.teams t /*таблица teams с алиасом t*/
JOIN sql.matches m ON t.api_id = m.home_team_api_id OR t.api_id = m.away_team_api_id /*оператор соединения inner JOIN; таблица teams с алиасом t; условие: home_team_api_id таблицы m равен api_id таблицы t или away_team_api_id таблицы m равен api_id таблицы t*/
```
И в таблице останется уже не 299 команд, а только 292. Дело в том, что таблица sql.matches по какой-то причине не содержит информацию о командах Lierse SK, KVC Westerlo, KAS Eupen, Club Brugge KV, KV Oostende, RSC Anderlecht и Hull City, зато они есть в таблице sql.teams. Возможно, эти команды не участвовали ни в одном матче или записи по этим матчам были удалены.

### **LEFT OUTER JOIN** И **RIGHT OUTER JOIN**
Также существуют схожие друг с другом типы соединения — LEFT JOIN и RIGHT JOIN (слово outer в операторе можно опустить).

Для *LEFT JOIN* работает следующее **правило**: из левой (относительно оператора) таблицы сохраняются все строки, а из правой добавляются только те, которые соответствуют условию соединения. Если в правой таблице не находится соответствия, то значения строк второй таблицы будут иметь значение **NULL**.

**LEFT JOIN** может быть полезен, когда соответствующих записей во второй таблице может не быть, но важно сохранить записи из первой таблицы.

Поставим следующую задачу: вывести полные названия команд, данных по которым нет в таблице matches.

Для начала посмотрим на результат запроса после соединения.
```sql
SELECT
    t.long_name, /*столбец long_name таблицы t*/
    m.id /*столбец id таблицы m*/
FROM sql.teams t /*таблица teams с алиасом t*/
LEFT JOIN sql.matches m ON t.api_id = m.home_team_api_id OR t.api_id = m.away_team_api_id /*оператор соединения left JOIN; таблица matches с алиасом m; условие: home_team_api_id таблицы m равен api_id таблицы t или away_team_api_id таблицы m равен api_id таблицы t*/
ORDER BY m.id DESC /*сортировка по id таблицы m по убыванию, чтобы увидеть строки со значением null*/
```
**Вывод**: в таблице *teams* сохранились все записи, а в таблице matches есть пустые строки.

Теперь, чтобы выбрать такие команды, которые не принимали участия в матчах, достаточно добавить условие *where m.id is null* (или любое другое поле таблицы *matches*).

```sql
SELECT
    t.long_name /*столбец long_name таблицы t*/
FROM 
    sql.teams t /*таблица teams с алиасом t*/
LEFT JOIN sql.matches m ON t.api_id = m.home_team_api_id OR t.api_id = m.away_team_api_id /*оператор соединения left JOIN; таблица matches с алиасом m; условие: home_team_api_id таблицы m равен api_id таблицы t или away_team_api_id таблицы m равен api_id таблицы t*/
WHERE m.id IS NULL /*условие: столбец id таблицы m имеет значение null*/
```
**Обратите внимание!** Если мы добавим какой-либо фильтр по значению для таблицы *matches*, то *LEFT JOIN* превратится в *INNER JOIN*, поскольку для второй таблицы станет необходимым присутствие значения в строке.

Используя LEFT JOIN, выведите список уникальных названий команд, содержащихся в таблице matches. Отсортируйте список в алфавитном порядке.
```sql
SELECT
   DISTINCT t.long_name
FROM sql.teams t  
LEFT JOIN sql.matches m ON t.api_id = m.home_team_api_id OR t.api_id = m.away_team_api_id
WHERE m.id is not null
ORDER BY t.long_name
```
С **LEFT JOIN** также работают **агрегатные функции**, что позволяет не потерять значения из левой таблицы. Например, мы можем вывести сумму голов команд по гостевым матчам.
```sql
SELECT
    t.long_name,
    SUM(m.away_team_goals) total_goals
FROM 
    sql.teams t /*таблица teams с алиасом t*/
LEFT JOIN sql.matches m ON t.api_id = m.away_team_api_id /*оператор соединения LEFT JOIN; таблица matches с алиасом m; условие: away_team_api_id таблицы m равен api_id таблицы t*/
GROUP BY t.id /*группировка по столбцу id таблицы t*/
ORDER BY total_goals DESC /*сортировка по столбцу total_goals по убыванию, чтобы увидеть строки со значением null*/
```
**Обратите внимание!** При применении функций SUM, MIN, MAX, AVG к полям со значением **NULL** в результате получится **NULL**, а не 0. А при использовании функции COUNT, наоборот, получится 0.

Используя LEFT JOIN, напишите запрос, который выведет полное название команды (long_name), количество матчей, в которых участвовала команда, — домашних и гостевых (matches_cnt). Отсортируйте по количеству матчей в порядке возрастания, затем — по названию команды в алфавитном порядке.
```sql
SELECT
    t.long_name,
    COUNT(m.id) matches_cnt
FROM
    sql.teams t
LEFT JOIN sql.matches m ON t.api_id = m.home_team_api_id or t.api_id = m.away_team_api_id
GROUP BY t.id
ORDER BY 2, 1
```
При использовании *RIGHT JOIN* сохраняется та же логика, что и для *LEFT JOIN*, только за основу берётся правая таблица.

Чтобы из *LEFT JOIN* получить *RIGHT JOIN*, нужно просто поменять порядок соединения таблиц.

Вообще, применение *RIGHT JOIN* считается **плохим тоном**, так как язык *SQL* читается и пишется слева направо, а такой оператор усложняет чтение запросов.

### **FULL OUTER JOIN**
Оператор *FULL OUTER JOIN* объединяет в себе *LEFT* и *RIGHT JOIN* и позволяет сохранить кортежи обеих таблиц. Даже если не будет соответствий, мы сохраним все записи из обеих таблиц.

**FULL OUTER JOIN** может быть полезен в ситуациях, когда схема данных недостаточно нормализована и не хватает таблиц-справочников.

Предположим, что вам необходимо получить полный список пользователей — и оформивших заказ, и зарегистрированных, — но в базе данных этой объединённой таблицы нет. В данном случае можно использовать FULL OUTER JOIN для получения полного списка, соединив таким образом таблицы c заказами и регистрациями по id пользователя.

Синтаксис FULL OUTER JOIN аналогичен другим JOIN.
```sql
SELECT 
…
FROM
    table1
FULL OUTER JOIN table2 ON условие
```

### **CROSS JOIN**

**CROSS JOIN** соединяет таблицы так, что каждая запись в первой таблице присоединяется к каждой записи во второй таблице, иначе говоря, даёт декартово произведение.
```sql
SELECT * /*выбор всех полей*/
FROM
    sql.teams, /*таблица teams*/
    sql.matches /*таблица matches*/
```
Этот же запрос можно записать с использованием CROSS JOIN.
```sql
SELECT * /*выбор всех полей*/
FROM
    sql.teams /*таблица teams*/
    CROSS JOIN sql.matches /*таблица matches*/
```
**Обратите внимание!** Условие для *CROSS JOIN*, в отличие от других операторов, не требуется. Также этот запрос можно записать с помощью INNER JOIN с условием on true — результат будет тот же.
```sql
SELECT * /*выбор всех полей*/
FROM
    sql.teams /*таблица teams*/
    JOIN sql.matches ON TRUE /*оператор соединения INNER JOIN; таблица matches; условие: для всех случаев*/
```
**CROSS JOIN** может быть полезен, когда необходимо создать таблицу фактов.

Например, с помощью такого запроса мы можем получить все возможные комбинации полных названий команд в матчах.
```sql
SELECT
    DISTINCT /*оператор исключения повторяющихся строк*/
    t1.long_name home_team, /*столбец long_name таблицы t1; новое название*/
    t2.long_name away_team /*столбец long_name таблицы t2; новое название*/
FROM
    sql.teams t1 /*таблица teams с алиасом t1*/
    CROSS JOIN sql.teams t2 /*оператор соединения CROSS JOIN; таблица teams с алиасом t2*/
```
Напишите запрос, который выведет все возможные уникальные комбинации коротких названий домашней команды (home_team) и коротких названий гостевой команды (away_team). Отсортируйте запрос по первому и второму столбцам.
```sql
SELECT 
    DISTINCT 
    t1.short_name home_team,
    t2.short_name away_team 
FROM 
    sql.teams t1 
    CROSS JOIN sql.teams t2
ORDER BY 1, 2
```
### **NATURAL JOIN**

Ключевое слово natural в начале оператора JOIN позволяет не указывать условие соединения таблиц — для соединения будут использованы столбцы с одинаковым названием из этих таблиц.

NATURAL JOIN можно использовать с любыми видами соединений, которые требуют условия соединения:

→ NATURAL INNER JOIN (возможна запись NATURAL JOIN);
→ NATURAL LEFT JOIN;
→ NATURAL RIGHT JOIN;
→ NATURAL FULL OUTER JOIN.

При использовании NATURAL JOIN прежде всего стоит обратить внимание на ключи таблиц. Для наших таблиц teams и matches этот вид соединения не подойдёт, так как общим для обеих таблиц является столбец id, но таблицы соединяются по другим столбцам.

Когда у таблиц есть несколько столбцов с одинаковыми именами, при NATURAL JOIN условие соединения будет применено на все столбцы с одинаковыми именами.

То есть для таблиц table1 и table2

table1: id, name, ...

table2: id, name, ...

запрос
```sql
SELECT 
…
FROM 
         table1 
NATURAL JOIN table2
```
будет равнозначен запросу
```sql
SELECT
…
FROM 
         table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id AND t1.name = t2.name
```
### **ОБЩАЯ ЛОГИКА ПОСТРОЕНИЯ ЗАПРОСА С JOIN**

При построении запроса с несколькими JOIN старайтесь идти слева направо. Сначала выберите таблицу, которая является центральной в соответствии с поставленной задачей, вопросом. Затем добавляйте таблицы поэтапно в зависимости от бизнес-логики запроса.

Например, для ответа на вопрос: «Какая команда сыграла больше всех матчей в сезоне 2010/2011?» в качестве центральной лучше выбрать таблицу с командами.

А для ответа на вопрос: «В каком сезоне участвовало больше всего команд?» — таблицу с матчами.

Стоит отметить, что из рассмотренных видов соединений чаще всего используются INNER JOIN и LEFT JOIN. Другие операторы используются реже, но стоит помнить об их существовании при решении нестандартных задач.

Напишите запрос, который выведет список уникальных полных названий команд (long_name), игравших в гостях в матчах сезона 2012/2013. Отсортируйте список в алфавитном порядке.
```sql
SELECT 
    DISTINCT 
    t.long_name long_name
FROM 
    sql.teams t 
    JOIN sql.matches m ON t.api_id = m.away_team_api_id
WHERE season = '2012/2013'
ORDER BY t.long_name
```
Напишите запрос, который выведет полное название команды (long_name) и общее количество матчей (matches_cnt), сыгранных командой Inter в домашних матчах.
```sql
SELECT 
    t.long_name long_name,
    COUNT(m.id) matches_cnt
FROM 
    sql.teams t 
    JOIN sql.matches m ON t.api_id = m.home_team_api_id
WHERE long_name = 'Inter'
GROUP BY t.long_name
```
ИЛИ можно так 
```sql
SELECT
    t.long_name,
    COUNT(m.id) matches_cnt
FROM sql.matches m
    JOIN sql.teams t ON t.api_id = m.home_team_api_id
WHERE t.long_name = 'Inter'
GROUP BY t.id
```
Напишите запрос, который выведет топ-10 команд (long_name) по суммарному количеству забитых голов в гостевых матчах. Во втором столбце запроса выведите суммарное количество голов в гостевых матчах (total_goals).
```sql
SELECT 
    t.long_name long_name,
    SUM(m.away_team_goals) total_goals
FROM 
    sql.teams t 
    JOIN sql.matches m ON t.api_id = m.away_team_api_id
GROUP BY t.long_name
ORDER BY SUM(m.away_team_goals) DESC
LIMIT 10
```
Выведите количество матчей между командами Real Madrid CF и FC Barcelona.
```sql
SELECT 
    COUNT(*)
FROM 
    sql.matches m 
    JOIN sql.teams h ON h.api_id = m.home_team_api_id 
    JOIN sql.teams a ON a.api_id = m.away_team_api_id
WHERE (h.long_name = 'Real Madrid CF' AND a.long_name = 'FC Barcelona')
    OR (h.long_name = 'FC Barcelona' AND a.long_name = 'Real Madrid CF')
```
Напишите запрос, который выведет название команды (long_name), сезон (season) и суммарное количество забитых голов в домашних матчах (total_goals). Оставьте только те строки, в которых суммарное количество голов менее десяти. Отсортируйте запрос по названию команды, а затем — по сезону.
```sql
SELECT 
    t.long_name,
    m.season,
    SUM(m.home_team_goals) total_goals
FROM 
    sql.teams t
    JOIN sql.matches m ON t.api_id = m.home_team_api_id
GROUP BY t.long_name, m.season
HAVING SUM(m.home_team_goals) < 10 
ORDER BY t.long_name, m.season
```