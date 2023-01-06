SQL относится к декларативным языкам, то есть SQL-запрос описывает ожидаемый результат, а не способ его получения.


## Последовательность выполнения операторов запроса
[Ресурс](https://techrocks.ru/2021/03/05/order-of-sql-operations/)

1. FROM (выбор таблицы) / JOIN (комбинация с подходящими по условию данными из второй таблицы)
2. WHERE (фильтрация строк)
3. GROUP BY (агрегирование данных)
4. HAVING (фильтрация агрегированных данных)
5. SELECT (возврат результирующего датасета) / CASE (if-else выражения)
6. DISTINCT
7. UNION (объединение )
8. ORDER BY (сортировка)
9. LIMIT, TOP OFFSET


## Шаблон запроса

```sql
SELECT
    EXTRACT(DAYOFWEEK FROM timestamp) AS day_of_week
    col1,
    col2,
    col3
    COUNT(id) AS num_of_rows -- MIN(), MAX(), AVG(), COUNT()  
FROM
    table
WHERE
    col1 = value  -- col1 != value
    AND
    col2 > a_val AND col2 < b_val -- col2 BETWEEN a_val AND b_val
    AND
    col3 IN (val1, val2, val3)

GROUP BY
    day_of_week,
    col1,
    col2,
    col3
HAVING
    num_of_rows > 10
    
ORDER BY
    num_of_rows DESC --ASC
```

### Как подсчитать количество строк?

```sql
COUNT(1)
```

## ORDER BY

```sql
SELECT 
    col1, col2, ...colN
FROM 
    tableName
WHERE 
    condition
ORDER BY 
    col1, col2, ...colN 
                        ASC - по возрастанию
                        DESC - по убыванию
;
```

## GROUP BY and HAVING

```sql
SELECT 
    col1, col2, ...colN
FROM 
    tableName
WHERE 
    condition

GROUP BY 
    col1, col2, ...colN; 
HAVING
    group_condition
```


## DATE and DATETIME, EXTRACT

```sql
DATE(timestamp)
EXTRACT(DATE from datetime) -- YEAR, MONTH, WEEK, DATE, DAYOFWEEK
TIMESTAMP_DIFF(date1, date2, SECOND)

TO_CHAR(timestamp, 'YYYY-MM-DD HH24:MI:SS')
```

### Example

```sql
sql = """
SELECT
    *
FROM
    table
WHERE
    timestamp::DATE = '2022-07-04'
LIMIT 5
```

## JOIN

INNER JOIN — возвращает записи, имеющиеся в обеих таблицах

LEFT JOIN — возвращает записи из левой таблицы, даже если такие записи отсутствуют в правой таблице

RIGHT JOIN — возвращает записи из правой таблицы, даже если такие записи отсутствуют в левой таблице

FULL JOIN — возвращает все записи объединяемых таблиц

CROSS JOIN — возвращает все возможные комбинации строк обеих таблиц

SELF JOIN — используется для объединения таблицы с самой собой


##  Операции над множествами - UNION, INTERSECT и EXCEPT

Оператор UNION используется для комбинации результатов двух и более инструкций SELECT.

UNION - вертикальная конкатенация
    ALL - возвращает все записи, включая дубликаты
    DISTINCT - возвращает только уникальные значения
    
INTERSECT - возвращает только строки зи первого SELECT, совпадающие со строками из второго SELECT.

EXCEPT / MINUS - возвращает только строки из первого SELECT, отсутствующие во втором SELECT
    



## Common table expression (CTE)

```sql
-- create CTE
WITH CTE_table AS (
    SELECT ...
    FROM ...
    WHERE ...
)

-- use CTE
SELECT ..
FROM  CTE_table
```


    

### Examples

```sql
WITH RelevantRides AS
(
   SELECT
       EXTRACT(HOUR FROM trip_start_timestamp) AS hour_of_day,
       trip_miles,
       trip_seconds


   FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
   WHERE
       trip_start_timestamp > '2017-01-01' AND trip_start_timestamp < '2017-07-01'
       AND 
       trip_seconds > 0
       AND
       trip_miles > 0
)

SELECT
    hour_of_day,
    COUNT(1) AS num_trips,
    3600 * SUM(trip_miles) / SUM(trip_seconds) AS avg_mph

FROM RelevantRides
GROUP BY
   hour_of_day
ORDER BY
   hour_of_day
```           
    
# Операции над строками

Здесь существуют различия между различными диалектами SQL.

## Ресурсы
- https://metanit.com/sql/sqlserver/8.1.php

## LIKE и регулярные выражения

```sql
WHERE
    string_cal LIKE "regular expression"
```

## Функции

Синтаксис функций отличается в различных диалектах SQL. Я заметил, что не все функции из приведенных ниже работают в PostgresSQL.


LEN() - возвращает количество символов в строке
```sql
SELECT LEN('Apple')  -- 5
```


LTRIM() - удаляет начальные пробелы в строке
```sql
SELECT LTRIM('  Apple')
```

RTRIM() - удаляет конечные пробелы из строки
```sql
SELECT RTRIM(' Apple    ')
```

CHARINDEX() - возвращает индекс, по которому находится первое вхождение подстроки в строке
```sql
SELECT CHARINDEX('pl', 'Apple') -- 3
```


PATINDEX() - возвращает индекс, по которому находится первое вхождение определенного шаблона в строке
```sql
SELECT PATINDEX('%p_e%', 'Apple')   -- 3
```


LEFT() - вырезает с начала строки определенное количество символов. Первый параметр функции - строка, а второй - количество символов, которые надо вырезать сначала строки
```sql
SELECT LEFT('Apple', 3) -- App
```

RIGHT() - вырезает с конца строки определенное количество символов. Первый параметр функции - строка, а второй - количество символов, которые надо вырезать сначала строки
```sql
SELECT RIGHT('Apple', 3)    -- ple
```

SUBSTRING() - вырезает из строки подстроку определенной длиной, начиная с определенного индекса. Певый параметр функции - строка, второй - начальный индекс для вырезки, и третий параметр - количество вырезаемых символов
```sql
SELECT SUBSTRING('Galaxy S8 Plus', 8, 2)    -- S8
```

REPLACE() - заменяет одну подстроку другой в рамках строки. Первый параметр функции - строка, второй - подстрока, которую надо заменить, а третий - подстрока, на которую надо заменить
```sql
SELECT REPLACE('Galaxy S8 Plus', 'S8 Plus', 'Note 8')   -- Galaxy Note 8
```

REVERSE() - переворачивает строку наоборот
```sql
SELECT REVERSE('123456789') -- 987654321
```

CONCAT() - объединяет две строки в одну. В качестве параметра принимает от 2-х и более строк, которые надо соединить
```sql
SELECT CONCAT('Tom', ' ', 'Smith')  -- Tom Smith
```

LOWER() - переводит строку в нижний регистр
```sql
SELECT LOWER('Apple')   -- apple
```

UPPER() - переводит строку в верхний регистр
```sql
SELECT UPPER('Apple')   -- APPLE
```

SPACE() - возвращает строку, которая содержит определенное количество пробелов

# Операции с множествами

## UNION

```sql
SELECT * FROM table_1
UNION
SELECT * FROM table_2
```

## UNION ALL

```sql
SELECT * FROM table_1
UNION ALL
SELECT * FROM table_2
```

## INTERSECT

## MINUS



# Оконные функции
## Ресурсы
- https://vc.ru/dev/130856-poleznye-okonnye-funkcii-sql

## Синтаксис

```sql
<название функции>(<выражение>) OVER (
    <окно>
  <сортировка>
  <границы окна>
)
```
```sql
SELECT *,
    AVG(time) OVER(
        PARTION BY id
        ORDER BY date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
        ) AS avg_time
FROM
```
### OVER

### PARTION

### ORDER BY

### window frame

```sql
ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
ROWS BETWEEN 15 PRECEDING AND 15 FOLLOWING
```

## Виды оконных функций

Оконные функции делятся на:
- Агрегатные функции
- Ранжирующие функции
- Функции смещения
- Аналитические функции

### Агрегатные функции - Aggregate functions

Собственно, те же, что и обычные, только встроенные в конструкцию с OVER: SUM, AVG, COUNT, MIN, MAX

### Ранжируемые функции - Numbering functions

ROW_NUMBER() — нумерует строки в результирующем наборе.

RANK() — присваивает ранг для каждой строки, если найдутся одинаковые значения, то следующий ранг присваивается с пропуском.

DENSE_RANK() — присваивает ранг для каждой строки, если найдутся одинаковые значения, то следующий ранг присваивается без пропуска.

NTILE() — помогает разделить результирующий набор на группы.

### Функции смещения / навигации -  Navigation functions

Используется в случаях, когда нужно обратиться к строке в наборе данных из окна с некоторым смещением относительно текущей строки.

LAG() — смещение назад

LEAD() — смещение вперед

LAG() и LEAD() имеют следующие аргументы:
    - Столбец, значение которого необходимо вернуть
    - На сколько строк выполнить смешение (дефолт =1)
    - Что вставить, если вернулся NULL
    

FIRST_VALUE() — найти первое значение набора данных

LAST_VALUE() — найти последнее значение набора данных





