# Medium

## Workers With The Highest Salaries

### Link
https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries?code_type=2

### Description
Find the titles of workers that earn the highest salary. 
Output the highest-paid title or multiple titles that share the highest salary.

### Solution
Иногда необходимо выделить записи с определенной числовой характеристикой (с минимальным, максимальным и прочим значением):
Во всех способах вначале мы формируем общую таблицу, чтобы затем выделить из нее нужные записи. 

#### PostgreSQL

В sql это можно сделать тремя способами:

1. Через подзапрос получить требуемое числовое значение, а затем отфильтровать по нему записи

```sql
select
      t.worker_title as title
from 
    worker w left join title t on 
        w.worker_id = t.worker_ref_id
where
    w.salary = (select max(salary) from worker)
```

2. CTE + Проставить числовые ранги с помощью оконной функции и отфильтровать записи по конкретному рангу.

```sql

with 
ranked_table as (
    select
          t.worker_title as title
        , w.salary
        , dense_rank() over (order by w.salary desc) as salary_rank
    from 
        worker w left join title t
            on w.worker_id = t.worker_ref_id
)
select title from ranked_table where salary_rank = 1

```


#### Pandas

```python
import pandas as pd

worker.head()
worker[worker.salary == worker.salary.max()].merge(title, how = 'inner', left_on = 'worker_id', right_on = 'worker_ref_id')[['worker_title']]
```


## Users By Average Session Time

### Link
https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=1

### Description

Calculate each user's average session time. 
A session is defined as the time difference between a page_load and page_exit. 

For simplicity, assume a user has only 1 session per day and if there are multiple of the same events on that day, 
consider only the latest page_load and earliest page_exit. 

Output the user_id and their average session time.

### Solution

#### PostgreSQL
```sql

WITH 
exit as (
    select
        user_id, 
        date(timestamp) as day, 
        min(timestamp) as exit
    from
        facebook_web_log
    where
        action = 'page_exit'
    group by 
        1,2
), 

load as (
    select
        user_id, 
        date(timestamp) as day, 
        max(timestamp) as load
    from
        facebook_web_log
    where
        action = 'page_load'
    group by
        1,2
)


SELECT
    exit.user_id, 
    avg(exit-load)
FROM
    exit left join load on 
        exit.user_id = load.user_id
        and 
        exit.day = load.day
GROUP BY 1

```

#### Pandas

```python
import pandas as pd
import numpy as np

df = pd.DataFrame(facebook_web_log)
df1 = df[df['action'] == 'page_load']
df1['day'] = df1['timestamp'].dt.date
df1 = df1.groupby(['user_id','day'], as_index = False).max()

df2 = df[df['action'] == 'page_exit']
df2['day'] = df2['timestamp'].dt.date
df2 = df2.groupby(['user_id','day'], as_index = False).min()

dfm = pd.merge(df1,df2,how = 'inner', on=['user_id','day'])
dfm['diff'] = dfm['timestamp_y'] - dfm['timestamp_x']
dfm.groupby('user_id',as_index = False).apply(np.mean)
```


## Finding User Purchases

### Link
https://platform.stratascratch.com/coding/10322-finding-user-purchases?code_type=1

### Description
Write a query that'll identify returning active users. A returning active user is a user that has made a second purchase within 7 days of any other of their purchases. Output a list of user_ids of these returning active users.

### Solution

Сила декартового произведения. Зачем учитывать порядок записей если можно посмотреть все их комбинации и отсеять те что идут не в хронологическом порядке - ведь их разность будет отрицательная!. 

#### PostgreSQL

```sql
SELECT 
    DISTINCT(a.user_id) 
FROM amazon_transactions a
JOIN amazon_transactions b
    ON a.user_id = b.user_id
WHERE a.created_at - b.created_at BETWEEN 0 AND 7
    AND a.id != b.id
```

#### Pandas


## Classify Business Type

### Link
https://platform.stratascratch.com/coding/9726-classify-business-type?code_type=1


### Description

Classify each business as either a restaurant, cafe, school, or other. A restaurant should have the word 'restaurant' in the business name. For cafes, either 'cafe', 'café', or 'coffee' can be in the business name. 'School' should be in the business name for schools. All other businesses should be classified as 'other'. Output the business name and the calculated classification.


### Solution

Хороший пример для практики case when then end.

#### PostgreSQL

```sql
select distinct
    business_name,
    (
        case
           when business_name ilike '%restaurant%' then 'restaurant'
           when business_name ilike '%school%' then 'school'
           when business_name ilike any(array['%cafe%', '%café%', '%coffee%']) then 'cafe'
           else 'other'
        end
    
    ) as business_type
from 
    sf_restaurant_health_violations
```

## Premium vs Freemium

### Link
    https://platform.stratascratch.com/coding/10300-premium-vs-freemium?code_type=1

### Description
Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads.

### Solution
Пример того как создавать сводную таблицу в sql. И подсчитывать значения с помощью sum + case.

#### PostgreSQL

```sql
with pivot_table as (
    select
        date,
        sum(case when paying_customer = 'yes' then downloads else 0 end) paying,
        sum(case when paying_customer = 'no' then downloads else 0 end) non_paying
    from 
        ms_download_facts d 
            full outer join ms_user_dimension u on d.user_id = u.user_id
            full outer join  ms_acc_dimension a on u.acc_id = a.acc_id
    
    group by date
    order by date
)

select * from pivot_table where non_paying > paying
```

## Acceptance Rate By Date

### Link
https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date?code_type=1

### Description

What is the overall friend acceptance rate by date? Your output should have the rate of acceptances by the date the request was sent. Order by the earliest date to latest.


Assume that each friend request starts by a user sending (i.e., user_id_sender) a friend request to another user (i.e., user_id_receiver) that's logged in the table with action = 'sent'. If the request is accepted, the table logs action = 'accepted'. If the request is not accepted, no record of action = 'accepted' is logged.


### Solution

Решение строиться на добавлении значений 'accepted' к записям 'sent', если о ни были. Сделать это можно с помощью join-ов. В моем первоначальном решении я делал join-ы с помощью под-запросов. Но в join-ы можно также добавлять условия, и тогда можно обойтись без под-запросов.

В решении pandas вышла оказия с методом .count(), когда я применил его к group внутри groupby.

#### PostgreSQL

```sql
select
      a.date
    , count(b.user_id_receiver)/count(a.user_id_sender)::float as acceptance_rate
from 
    fb_friend_requests a left join fb_friend_requests b
        on a.user_id_sender = b.user_id_sender
        and b.action = 'accepted'
where a.action='sent'
group by a.date
```

#### Pandas

```python
fb_friend_requests.merge(
    fb_friend_requests[fb_friend_requests['action'] == 'accepted'], 
    on='user_id_sender',
    suffixes=('_left', '_right'),
    how='left'
).loc[fb_friend_requests.action=='sent'].groupby('date_left').apply(lambda group: group[group['action_right'] == 'accepted'].shape[0]/group.shape[0]).reset_index()
```

## Task

### Link


### Description
Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.

### Solution

Здесь следует обратить внимание на оператор union all. Вначале я написал просто union и пересекающиеся строки были исключены. Но в этой задаче их нужно учесть, поэтому следует использовать union all.

#### PostgreSQL

```sql
with full_table as (
    select * from fb_eu_energy
    union all
    select * from fb_asia_energy
    union all
    select * from fb_na_energy
)

select
      date
    , sum(consumption) as sum_consumption
from full_table
group by date
order by sum_consumption desc
limit 1
```

## Highest Target Under Manager

### Link
https://platform.stratascratch.com/coding/9905-highest-target-under-manager?code_type=1

### Description
Find the highest target achieved by the employee or employees who works under the manager id 13. Output the first name of the employee and target achieved. The solution should show the highest target achieved under manager_id=13 and which employee(s) achieved it.

### Solution
Чтобы выделить людей, достигших максимального target-а, можно составить подзапрос, или добавить новый признак через rank.
Я добавил оба решения. 
Также добавил наработку с использованием dense_rank(). В отличае от rank(), он выставляет нумерацию по порядку (1, 2, 3, ...), а не по номеру строки, на которой закончился предыдущий ранг.

#### PostgreSQL

```sql
-- solution №1
-- with rank_table as (
--     select 
--           first_name
--         , target
--         , rank() over (order by target desc)
--     from salesforce_employees
--     where manager_id = 13
-- )

-- select
--       first_name
--     , target
-- from rank_table
-- where rank = 1

-- solution №2
select
      first_name
    , target
from salesforce_employees
where
    target = (select max(target) from salesforce_employees where manager_id = 13)
    and
    manager_id = 13



-- with rank_table as (
--     select 
--           first_name
--         , target
--         , rank() over (partition by manager_id order by target desc) as target_rank
--         , dense_rank() over (order by manager_id desc) as manager_rank
--     from salesforce_employees
-- )

-- select
--       first_name
--     , target
--     , *
-- from rank_table
-- order by manager_rank
```


## Highest Salary In Department

### Link
https://platform.stratascratch.com/coding/9897-highest-salary-in-department?code_type=1

### Description
Find the employee with the highest salary per department.
Output the department name, employee's first name along with the corresponding salary.

### Solution

Как вывести максимальное значение в определенной группе?

#### PostgreSQL

```sql
-- solution №1
-- with max_salary_table as (
--     select
--           department
--         , max(salary) as max_dep_salary
--     from employee
--     group by department
-- )


-- select
--       empl.department
--     , empl.first_name
--     , empl.salary
-- from
--     max_salary_table left join employee empl on 
--         empl.department = max_salary_table.department
--         and
--         empl.salary = max_salary_table.max_dep_salary
           
-- solution №2
select
      department
    , first_name
    , salary
    
from employee
where (department, salary) in (
    select
          department
        , max(salary) as max_dep_salary
    from employee
    group by department
)
```

## Reviews of Categories

### Link
https://platform.stratascratch.com/coding/10049-reviews-of-categories?code_type=1

### Description
Find the top business categories based on the total number of reviews. Output the category along with the total number of reviews. Order by total reviews in descending order.

### Solution

Как разделить составное значение колонки на несколько строк?
Как работает решение №2.  


#### PostgreSQL

```sql
-- solution №1
select distinct
      category
    , sum(review_count) over (partition by category) as total_reviews_number
from 
    yelp_business, unnest(string_to_array(categories, ';')) category
order by 
    total_reviews_number desc


-- solution №2
select 
      unnest(string_to_array(categories, ';')) as category
    , sum(review_count) as total_reviews
from
    yelp_business
group by category
order by total_reviews desc
```

## Find matching hosts and guests in a way that they are both of the same gender and nationality

### Link
https://platform.stratascratch.com/coding/10078-find-matching-hosts-and-guests-in-a-way-that-they-are-both-of-the-same-gender-and-nationality?code_type=1

### Description
Find matching hosts and guests pairs in a way that they are both of the same gender and nationality.
Output the host id and the guest id of matched pair.

### Solution

В данном случае можно использовать как join, так и cross join. Но какой способ быстрее?

#### PostgreSQL

```sql
-- solution №1
select distinct
      h_tab.host_id
    -- , h_tab.nationality as host_nation
    -- , h_tab.gender as host_gender
    
    , g_tab.guest_id
    -- , g_tab.nationality as guest_nation
    -- , g_tab.gender as guest_gender
from 
    airbnb_hosts h_tab cross join airbnb_guests g_tab
where
        h_tab.nationality = g_tab.nationality
    and h_tab.gender = g_tab.gender
    
-- solution №2
select distinct
      h_tab.host_id
    -- , h_tab.nationality as host_nation
    -- , h_tab.gender as host_gender
    
    , g_tab.guest_id
    -- , g_tab.nationality as guest_nation
    -- , g_tab.gender as guest_gender
from 
    airbnb_hosts h_tab join airbnb_guests g_tab on
            h_tab.nationality = g_tab.nationality
        and h_tab.gender = g_tab.gender
```

#### Pandas

## Number Of Units Per Nationality

### Link
https://platform.stratascratch.com/coding/10156-number-of-units-per-nationality?code_type=1

### Description
Find the number of apartments per nationality that are owned by people under 30 years old.
Output the nationality along with the number of apartments.
Sort records by the apartments count in descending order.

### Solution
Почему при левом объединении (left join) появляется много неуникальных строк?

#### PostgreSQL

```sql
-- solution №1 
-- why do a lot of non-unique strings appear after left joining
select
    h_tab.nationality
    , count(distinct u_tab.unit_id) as n_apartment
from
    airbnb_units u_tab left join airbnb_hosts h_tab  on
        u_tab.host_id = h_tab.host_id 
where
        h_tab.age < 30
    and u_tab.unit_type = 'Apartment'
    
group by h_tab.nationality
order by n_apartment
```

## Ranking Most Active Guests

### Link
https://platform.stratascratch.com/coding/10159-ranking-most-active-guests?code_type=1

### Description
Rank guests based on the number of messages they've exchanged with the hosts. Guests with the same number of messages as other guests should have the same rank. Do not skip rankings if the preceding rankings are identical.
Output the rank, guest id, and number of total messages they've sent. Order by the highest number of total messages first.

### Solution
Эта задача хороший пример одновременного использования группировки и оконной функции.

Здесь необходимо просумировать значения n_messages по id_guest, прежде чем выставлять ранги. Однако, в условии это явно не праписано.
Следовательно, перед непосредственным решением (или хотя бы перед его сдачей), необходимо прикинуть, какие нюансы могут быть в данных, из за которых ответ может измениться. И учесть их.

#### PostgreSQL

```sql
-- check if there are several rows for the same id_guest - True - need to sum up
select
      id_guest
    , count(id_guest) c
from
    airbnb_contacts
group by
    id_guest
order by c desc


-- solution №1
with summed_up_table as (
    select id_guest, sum(n_messages) as sum_n_messages
    from airbnb_contacts
    group by id_guest
)

select
      dense_rank() over (order by sum_n_messages desc) as rank
    , id_guest
    , sum_n_messages
from
    summed_up_table
order by
    sum_n_messages desc

-- solution №2
select
      dense_rank() over (order by sum(n_messages) desc) as rank
    , id_guest
    , sum(n_messages) as sum_n_messages
from
    airbnb_contacts
group by 
    id_guest
order by 
    sum_n_messages desc
```

## Income By Title and Gender

### Link
https://platform.stratascratch.com/coding/10077-income-by-title-and-gender?code_type=1

### Description
Find the average total compensation based on employee titles and gender. Total compensation is calculated by adding both the salary and bonus of each employee. However, not every employee receives a bonus so disregard employees without bonuses in your calculation. Employee can receive more than one bonus.
Output the employee title, gender (i.e., sex), along with the average total compensation.

### Solution
В этом задании очень важно внимательно прочитать условие и не брасаться его решать.
Помимо этого, здесь интересно одновременное использование агрегирующей функции avg() и оператора case.

#### PostgreSQL

```sql
with summed_bonus as (
    select
          worker_ref_id as id
        , sum(bonus) as sum_bonus
    from
        sf_bonus
    group by 
        worker_ref_id
)

select 
      e.employee_title
    , e.sex
    , avg(e.salary + (case when b.sum_bonus is not null then b.sum_bonus else 0 end)) as avg_compensation

from 
    sf_employee e inner join summed_bonus b on
        e.id = b.id
group by
      e.employee_title 
    , e.sex
order by
      e.employee_title 
    , e.sex
```

## Find all wineries which produce wines by possessing aromas of plum, cherry, rose, or hazelnut

### Link
https://platform.stratascratch.com/coding/10026-find-all-wineries-which-produce-wines-by-possessing-aromas-of-plum-cherry-rose-or-hazelnut?code_type=1

### Description
Find all wineries which produce wines by possessing aromas of plum, cherry, rose, or hazelnut. To make it more simple, look only for singular form of the mentioned aromas.
Output unique winery values only.

### Solution
Как проверить есть ли в строке хотя бы одна подстрока из массива?

#### PostgreSQL

```sql
select distinct
      winery
from 
    winemag_p1
where 
    lower(description) similar to '%(plum|cherry|rose|hazelnut)%'
    -- description ilike '%(plum|cherry|rose|hazelnut)%'
```

# Hard

## Monthly Percentage Difference

### Link
https://platform.stratascratch.com/coding/10319-monthly-percentage-difference?code_type=2

### Description

Given a table of purchases by date, calculate the month-over-month percentage change in revenue. 

The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.

The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)*100.


### Solution

#### PostgreSQL

```sql
SELECT
    TO_CHAR(created_at::date, 'YYYY-MM') as year_month,
    ROUND((SUM(value) - LAG(SUM(value), 1) OVER ())*100/LAG(SUM(value), 1) OVER (), 2) AS revenue_diff
    
FROM
    sf_transactions
GROUP BY year_month
ORDER BY 
    year_month
    
```

```sql
SELECT 
    to_char(created_at::date, 'YYYY-MM') as year_month,
    round((sum(value) - lag(sum(value),1) over (w)) / lag(sum(value),1) over (w)*100, 2) as revenue_diff
FROM 
    sf_transactions
GROUP BY 
    year_month
WINDOW w as (order by to_char(created_at::date, 'YYYY-MM'))
```

#### Pandas

```python
import pandas as pd

sf_transactions.head()
res = sf_transactions.set_index('created_at')['value'].resample('M').sum().pct_change()*100
res.round(2).to_period().reset_index()
```


## 

### Link

### Description


### Solution

#### PostgreSQL

#### Pandas