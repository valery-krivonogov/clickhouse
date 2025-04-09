#  Выполнение домашнего задания:

## Развернуть кластер Cassandra

Задание:<br>

+ *развернуть БД;*<br>
+ *выполнить импорт тестовой БД;*<br>
+ *выполнить несколько запросов и оценить скорость выполнения.*<br>
+ *развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов *<br>
+ *развернуть Кликхаус в кластерном исполнении, создать распределенную таблицу, заполнить данными и протестировать скорость по сравнению с 1 инстансом *<br>

Сервер развертывается на одной ноде: clickhouse-server <br>

Развертывание на сервере с ОС Windows-10<br>
Версия образа clickhouse:latest<br>

docker-compose.yml:
```
version: "3.8"

services:
  clickhouse-server:
    image: clickhouse:latest
    container_name: clickhouse-server
    ports:
      - 9000:9000
      - 8123:8123
    environment:
      - CLICKHOUSE_DB=my_database
      - CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1 
      - CLICKHOUSE_USER=valery
      - CLICKHOUSE_PASSWORD=123456
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
    cap_add:
      - SYS_NICE
      - NET_ADMIN
      - IPC_LOCK
      - SYS_PTRACE
```

Команда запуска:
docker-compose -f docker-compose.yml up <br/>

Лог сервера:<br/>
```
/entrypoint.sh: create new user 'valery' instead 'default'
Processing configuration file '/etc/clickhouse-server/config.xml'.
Merging configuration file '/etc/clickhouse-server/config.d/docker_related_config.xml'.
Logging trace to /var/log/clickhouse-server/clickhouse-server.log
Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log

/entrypoint.sh: create database 'my_database'
Processing configuration file '/etc/clickhouse-server/config.xml'.
Merging configuration file '/etc/clickhouse-server/config.d/docker_related_config.xml'.
Logging trace to /var/log/clickhouse-server/clickhouse-server.log
Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
```

Подключение к развернотому серверу через clickhouse-client: <br/> 
```
root@docker-desktop:/# clickhouse-client
ClickHouse client version 25.3.2.39 (official build).
Connecting to localhost:9000 as user valery.
Connected to ClickHouse server version 25.3.2.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. You can enable it using `echo 1 > /proc/sys/kernel/task_delayacct` or by using sysctl.
 * Linux is not using a fast clock source. Performance can be degraded. Check /sys/devices/system/clocksource/clocksource0/current_clocksource
 * Linux transparent hugepages are set to "always". Check /sys/kernel/mm/transparent_hugepage/enabled

docker-desktop :) SELECT * FROM system.databases;

SELECT *
FROM system.databases

Query id: 6617f207-e33f-45f8-8e52-874b3857dad6

   ┌─name───────────────┬─engine─┬─data_path──────────────────┬─metadata_path───────────────────────────────────┬─uuid─────────────────────────────────┬─engine_full─┬─comment─┐
1. │ INFORMATION_SCHEMA │ Memory │ /var/lib/clickhouse/       │                                                 │ 00000000-0000-0000-0000-000000000000 │ Memory      │         │
2. │ default            │ Atomic │ /var/lib/clickhouse/store/ │ store/b43/b430fec3-c016-4083-b1ed-ababc8e108d9/ │ b430fec3-c016-4083-b1ed-ababc8e108d9 │ Atomic      │         │
3. │ information_schema │ Memory │ /var/lib/clickhouse/       │                                                 │ 00000000-0000-0000-0000-000000000000 │ Memory      │         │
4. │ my_database        │ Atomic │ /var/lib/clickhouse/store/ │ store/15e/15e51100-283c-47cc-951c-c73b915a0fd6/ │ 15e51100-283c-47cc-951c-c73b915a0fd6 │ Atomic      │         │
5. │ system             │ Atomic │ /var/lib/clickhouse/store/ │ store/d9d/d9d13cb7-654d-4431-b115-48e3590f291d/ │ d9d13cb7-654d-4431-b115-48e3590f291d │ Atomic      │         │
   └────────────────────┴────────┴────────────────────────────┴─────────────────────────────────────────────────┴──────────────────────────────────────┴─────────────┴─────────┘

5 rows in set. Elapsed: 0.004 sec.

docker-desktop :)
```

### Создание БД , наполнение небольшими тестовыми данными

Реализация:<br>

```
docker-desktop :)
```


### Импорт тестовой БД, нагрузочное тестирование
Импортировать тестовую БД OnTime (https://clickhouse.com/docs/getting-started/example-datasets/ontime):<br>



```
-- загрузка данных в файловую систему сервера (папка tmp) 
root@5ce3ceb8e8a9:/# cd tmp
root@5ce3ceb8e8a9:/tmp# ls

root@5ce3ceb8e8a9:/tmp# wget --no-check-certificate --continue https://transtats.bts.gov/PREZIP/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_{1987..2022}_{1..12}.zip
--2025-04-09 08:42:39--  https://transtats.bts.gov/PREZIP/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_1.zip
Resolving transtats.bts.gov (transtats.bts.gov)... 204.68.194.70
Connecting to transtats.bts.gov (transtats.bts.gov)|204.68.194.70|:443... connected.

--2025-04-09 08:42:42--  https://transtats.bts.gov/PREZIP/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_10.zip
Reusing existing connection to transtats.bts.gov:443.
HTTP request sent, awaiting response... 200 OK
Length: 8162125 (7.8M) [application/x-zip-compressed]
Saving to: ‘On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_10.zip’

On_Time_Reporting_Carrier_On_Time_Perform 100%[=====================================================================================>]   7.78M   776KB/s    in 11s     

2025-04-09 08:42:53 (749 KB/s) - ‘On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_10.zip’ saved [8162125/8162125]
...

-- установка unzip (вывод команд не приводится)

root@5ce3ceb8e8a9:/tmp# apt-get update
root@5ce3ceb8e8a9:/tmp# apt-get install unzip

-- распаковка и загрузка файлов по списку (вывод команды сокращен)

root@5ce3ceb8e8a9:/tmp# ls -1 *.zip | xargs -I{} -P $(nproc) bash -c "echo {}; unzip -cq {} '*.csv' | sed 's/\.00//g' | clickhouse-client --input_format_csv_empty_as_default 1 --query='INSERT INTO ontime FORMAT CSVWithNames'"
On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_10.zip
On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1987_11.zip
...
On_Time_Reporting_Carrier_On_Time_Performance_1987_present_1995_7.zip

```


Проверить скорость выполнения запросов

```
SELECT avg(c1)
FROM
(
    SELECT
        Year,
        Month,
        count(*) AS c1
    FROM ontime
    GROUP BY
        Year,
        Month
)

Query id: 02dc5713-bc14-4d03-bf19-8d176ca74c45

   ┌───────────avg(c1)─┐
1. │ 429458.3829787234 │
   └───────────────────┘

1 row in set. Elapsed: 0.086 sec. Processed 40.37 million rows, 121.11 MB (469.63 million rows/s., 1.41 GB/s.)
Peak memory usage: 214.30 KiB.


SELECT
    DayOfWeek,
    count(*) AS c
FROM ontime
WHERE (Year >= 1990) AND (Year <= 2000)
GROUP BY DayOfWeek
ORDER BY c DESC

Query id: 98b127fd-86c5-47c2-83d3-77d41356f924

   ┌─DayOfWeek─┬───────c─┐
1. │         1 │ 4234816 │
2. │         2 │ 4226919 │
3. │         3 │ 4225158 │
4. │         4 │ 4198866 │
5. │         5 │ 4192171 │
6. │         7 │ 3967764 │
7. │         6 │ 3768272 │
   └───────────┴─────────┘

7 rows in set. Elapsed: 0.260 sec. Processed 28.82 million rows, 86.45 MB (110.92 million rows/s., 332.76 MB/s.)
Peak memory usage: 130.10 KiB.


```

