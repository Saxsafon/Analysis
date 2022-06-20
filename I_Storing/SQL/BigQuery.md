

client = bigquery.Client() - создание клиента

dataset_ref = client.dataset("hacker_news", project="bigquery-public-data") - создание ссылки на публичный датасет

dataset = client.get_dataset(dataset_ref) - загрузка датасета


client.list_tables(dataset) - получить список таблиц в датасете
for table in client.list_tables(dataset):
    print(table.full_table_id) - вывод полного идентификатора таблицы
    print(table.table_id) - вывод названия таблицы


table_ref = dataset_ref.table("full") - создание ссылки на конкретную таблицу
table = client.get_table(table_ref) - загрузка таблицы

table.schema - получить схему таблицы

client.list_rows(table, max_results=5).to_dataframe() - получить список строк в конкретной таблице

query = 'SELECT ... FROM ... WHERE ...'
query_job = client.query(query)

query_job.to_dataframe()



### Создать конфиг запуска, чтобы оценить размер возвращаемых запросом данных без его запуска 

dry_run_config = bigquery.QueryJobConfig(dry_run=True) - измерить объем данных, возвращаемых запросом, без его выполнения

dry_run_query_job = client.query(query, job_config=dry_run_config) - "сухой" запуск запроса


ONE_MB = 1000*1000
safe_config = bigquery.QueryJobConfig(maximum_bytes_billed=ONE_MB)
safe_query_job = client.query(query, job_config=safe_config) - Запрос с ограничением на количество сканируемых данных

