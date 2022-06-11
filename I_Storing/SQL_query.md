## Basic query template

'''
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
'''

### Как подсчитать количество строк?

COUNT(1)


## DATE and DATETIME, EXTRACT

DATE()
EXTRACT(DATE from datetime) -- YEAR, MONTH, WEEK, DATE, DAYOFWEEK
TIMESTAMP_DIFF(date1, date2, SECOND)

## Concat

UNION - vertical concatination
    ALL
    DISTINCT

## Common table expression (CTE)


-- create CTE
WITH CTE_table AS (
    SELECT ...
    FROM ...
    WHERE ...
)

-- use CTE
SELECT ..
FROM  CTE_table



    

## Examples

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
               
    
## String processing

LIKE