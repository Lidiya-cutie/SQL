# GROUP BY

Для вывода различных групп строк воспользуемся ключевым словом GROUP BY.
Примеры использования:
```sql
SELECT pet
    COUNT(quaqntity)
FROM for_aggergation
GROUP BY pet
```
```sql
SELECT pet
    SUM(quaqntity)
FROM for_aggergation
GROUP BY pet
```
```sql
SELECT pet
    AVG(quaqntity)
FROM for_aggergation
GROUP BY pet
```
```sql
SELECT pet
    MAX(quaqntity)
FROM for_aggergation
GROUP BY pet
```
*GROUP BY* используется для определения групп выходных строк, к которым могут применяться агрегатные функции.

Выведем число покемонов каждого типа.
```SQL
SELECT /*выбор*/
    type1 AS pokemon_type, /*столбец type1; присвоить алиас pokemon_type*/
    COUNT(*) AS pokemon_count /*подсчёт всех строк; присвоить алиас pokemon_count*/
FROM sql.pokemon /*из таблицы sql.pokemon*/
GROUP BY type1 /*группировка по столбцу type1*/
ORDER BY type1 /*сортировка по столбцу type1*/
```
Вывод, конечно же, можно сортировать по столбцу с агрегированием.

Представим ТОП существующих типов покемонов.
```sql
SELECT /*выбор*/
    type1 AS pokemon_type, /*столбец type1; присвоить алиас pokemon_type*/
    COUNT(*) AS pokemon_count /*подсчёт всех строк; присвоить алиас pokemon_count*/
FROM sql.pokemon /*из таблицы sql.pokemon*/
GROUP BY pokemon_type /*группировка по столбцу pokemon_type*/
ORDER BY COUNT(*) DESC /*сортировка в порядке убывания*/
```
**Обратите внимание!** Мы использовали в группировке не название столбца, а его алиас.

Напишите запрос, который выведет:

* число различных дополнительных типов (столбец additional_types_count),
* среднее число очков здоровья (столбец avg_hp),
* сумму показателей атаки (столбец attack_sum) в разбивке по основным типам (столбец primary_type).

Отсортируйте результат по числу дополнительных типов в порядке убывания, при равенстве — по основному типу в алфавитном порядке. Столбцы к выводу (обратите внимание на порядок!): primary_type, additional_types_count, avg_hp, attack_sum.
```SQL
SELECT
  type1 primary_type,
  COUNT(DISTINCT type2) additional_types_count,
  AVG(hp) avg_hp,
  SUM(attack) attack_sum
FROM sql.pokemon
GROUP BY primary_type
ORDER BY primary_type, additional_types_count DESC
```
Мы можем осуществлять группировку по нескольким столбцам.
```SQL
SELECT /*выбор*/
    type1 AS primary_type, /*столбец type1; присвоить алиас primary_type*/
    type2 AS additional_type, /*столбец type2; присвоить алиас additional_type*/
    COUNT(*) AS pokemon_count /*подсчёт всех строк присвоить алиас pokemon_count*/
FROM sql.pokemon /*из таблицы sql.pokemon*/
GROUP BY 1, 2 /*группировка по столбцам 1 и 2*/
ORDER BY 1, 2 NULLS FIRST /*сортировка по столбцам 1 и 2; сначала нули*/
```
**Обратите внимание!** В группировке можно указывать порядковый номер столбца так же, как мы делали это в прошлом модуле для сортировки.

*GROUP BY* можно использовать и без агрегатных функций. Тогда его действие будет равносильно действию *DISTINCT*.

Сравните выводы двух запросов:
```SQL
SELECT DISTINCT 
    type1
FROM sql.pokemon
```
```SQL
SELECT
    type1
FROM sql.pokemon
GROUP BY type1
```
Если ключевое слово *WHERE* определяет фильтрацию строк до агрегирования, то для фильтрации уже агрегированных данных применяется ключевое слово *HAVING*.

Например:
```sql
SELECT pet,
    COUNT(*)
FROM for_aggergation
GROUP BY pet
HAVING COUNT(quantity) < 1
```
**Важно!** *HAVING* обязательно пишется после *GROUP BY*.

Стоит различать следующее:
```sql
SELECT COUNT(pet) /*подсчёт всех строк pet*/
FROM for_aggergation
```
и
```sql
SELECT COUNT(DISTINCT pet) /*подсчёт уникальных строк pet*/
FROM for_aggergation
```
Напишем запрос про покемонов, в котором количество покемонов больше 50
```sql
SELECT
    type1
    COUNT(*)
FROM sql.pokemon
GROUP BY type1
HAVING COUNT(*)
ORDER BY type1
```
Выведем типы покемонов и их средний показатель атаки, при этом оставим только тех, у кого средняя атака больше 90.
```sql
SELECT /*выбор*/
    type1 AS primary_type, /*таблица type1; присвоить алиас primary_type*/
    AVG(attack) AS avg_attack /*расчёт среднего по столбцу attack; присвоить алиас avg_attack*/
FROM sql.pokemon /*из таблицы sql.pokemon*/
GROUP BY primary_type /*группировать по столбцу primary_type*/
HAVING AVG(attack) > 90 /*фильтровать по среднему значению attack, превышающему 90*/
```
Можно отсортировать по среднему показателю атаки:
```sql
SELECT /*выбор*/
    type1 AS primary_type, /*таблица type1; присвоить алиас primary_type*/
    AVG(attack) AS avg_attack /*расчёт среднего по столбцу attack; присвоить алиас avg_attack*/
FROM sql.pokemon /*из таблицы sql.pokemon*/
GROUP BY primary_type /*группировать по столбцу primary_type*/
HAVING AVG(attack) > 90 /*фильтровать по среднему значению attack, превышающему 90*/
ORDER BY AVG(attack) /*сортировать по среднему значению attack*/
```
Попробуйте удалить из запроса вывод второго столбца (со средним показателем атаки).

Запрос работает и выводит только названия типов, у которых средний показатель атаки выше 90.

Напишите запрос, который выведет основной и дополнительный типы покемонов (столбцы primary_type и additional_type) для тех, у кого средний показатель атаки больше 100 и максимальный показатель очков здоровья меньше 80.
```sql
SELECT
  type1 primary_type,
  type2 additional_type
FROM sql.pokemon
GROUP BY primary_type, additional_type
HAVING AVG(attack) > 100 
    AND MAX(hp) < 80
```
Напишите запрос, чтобы для покемонов, чьё имя (name) начинается с S, вывести столбцы с их основным типом (primary_type) и общим числом покемонов этого типа (pokemon_count). Оставьте только те типы, у которых средний показатель защиты больше 80. Выведите топ-3 типов по числу покемонов в них.
```sql
SELECT
  type1 primary_type,
  COUNT(*) pokemon_count
FROM sql.pokemon
WHERE name LIKE 'S%' 
GROUP BY  primary_type
HAVING AVG(defense) > 80 
ORDER BY COUNT(*) DESC
LIMIT 3
```
Для того, чтобы вычислить количество различных значений показателей атаки есть у покемонов с типом Water (основным или дополнительным), необходимо агрегировать с помощью count()
```sql
SELECT 
    count(distinct attack)
FROM sql.pokemon
where type2='Water' or type1='Water'
```
Напишите запрос, который выведет основной и дополнительный типы покемонов и средние значения по каждому показателю (столбцы avg_hp, avg_attack, avg_defense, avg_speed).Оставьте только те пары типов, у которых сумма этих четырёх показателей более 400.
```sql
SELECT 
  type1,
  type2,
  AVG(hp) avg_hp,
  AVG(attack) avg_attack,
  AVG(defense) avg_defense,
  AVG(speed) avg_speed
FROM sql.pokemon
GROUP BY type1, type2
HAVING AVG(hp)+AVG(attack)+AVG(defense)+AVG(speed)>400
```
Напишите запрос, который выведет столбцы с основным типом покемона и общим количеством покемонов этого типа. Учитывайте только тех покемонов, у кого или показатель атаки, или показатель защиты принимает значение между 50 и 100 включительно. Оставьте только те типы покемонов, у которых максимальный показатель здоровья не больше 125. Выведите только тот тип, который находится на пятом месте по количеству покемонов.
```SQL
SELECT 
  type1,
  COUNT(*)
FROM sql.pokemon
WHERE attack BETWEEN 50 AND 100 or defense BETWEEN 50 AND 100
GROUP BY type1
HAVING MAX(hp) <= 125
ORDER BY COUNT(*) DESC
OFFSET 4 LIMIT 1

```

Об отличиях *HAVING* от *WHERE* можно прочитать в [официальной документации](https://postgrespro.ru/docs/postgresql/11/tutorial-agg).
## ВМЕСТО РЕЗЮМЕ

В общем виде синтаксис оператора *SELECT*, с учётом имеющихся на данный момент знаний, представляем следующим образом:
```sql
SELECT [ALL | DISTINCT] список_столбцов|*
FROM список_имён_таблиц
[WHERE условие_поиска]
[GROUP BY список_имён_столбцов]
[HAVING условие_поиска]
[ORDER BY имя_столбца [ASC | DESC],…]
```
**Обратите внимание!** В квадратных скобках указаны необязательные предложения: они могут отсутствовать в операторе *SELECT*.

И в довершение итогов напомним структуру запроса, который мы можем составить с учётом новых знаний:
```SQL
SELECT
    столбец1 AS новое_название,
    столбец2,
    АГРЕГАТ(столбец3)
FROM таблица
WHERE (условие1 OR условие2)
    AND условие3
GROUP BY столбец1, столбец2
HAVING АГРЕГАТ(столбец3) > 5
ORDER BY сортировка1, сортировка2
OFFSET 1 LIMIT 2
```
Для проверки запроса можно не удалять весь запрос или его часть, а закомментиировать, например так:
```sql
SELECT pet
-- COUNT(quantity)
FRON for_aggergation
GROUP BY pet
```