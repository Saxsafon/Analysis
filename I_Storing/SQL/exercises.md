
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

SELECT 
    to_char(created_at::date, 'YYYY-MM') as year_month,
    round((sum(value) - lag(sum(value),1) over (w)) / lag(sum(value),1) over (w)*100, 2) as revenue_diff
FROM 
    sf_transactions
GROUP BY 
    year_month
WINDOW w as (order by to_char(created_at::date, 'YYYY-MM'))

#### Pandas

import pandas as pd

sf_transactions.head()
res = sf_transactions.set_index('created_at')['value'].resample('M').sum().pct_change()*100
res.round(2).to_period().reset_index()


# Medium

## Workers With The Highest Salaries

### Link
https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries?code_type=2

### Description
Find the titles of workers that earn the highest salary. 
Output the highest-paid title or multiple titles that share the highest salary.

### Solution

#### PostgreSQL

SELECT
    worker_title as title
FROM 
    worker left join title on 
        worker_id = worker_ref_id
WHERE 
    salary=(SELECT MAX(salary) FROM worker)

#### Pandas

import pandas as pd

worker.head()

worker[worker.salary == worker.salary.max()].merge(title, how = 'inner', left_on = 'worker_id', right_on = 'worker_ref_id')[['worker_title']]



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
    exit left join load 
        on 
            exit.user_id = load.user_id
            and 
            exit.day = load.day
GROUP BY 1

#### Pandas

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



##

### Link

### Description

### Solution

#### PostgreSQL

#### Pandas
