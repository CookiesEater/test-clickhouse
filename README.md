1. Создание необходимых таблиц
    ```sql
    CREATE DATABASE `log`;
    CREATE TABLE `hits` (
        `country` String,
        `ip` FixedString(15),
        `datetime` DateTime,
        `url` String
    ) ENGINE = MergeTree()
    PARTITION BY toYYYYMM(`datetime`)
    ORDER BY (`ip`, `datetime`)
    ```

2. Механизм сохранения в ClickHouse логов посещаемости сайта

    Раз в час (или другой период) парсить логи от nginx с добавлением необходимой информации (например, определение страны по ip через GeoIp БД) и запись данной информации пачкой в ClickHouse
    
3. Запрос на получение статистики по посещаемости сайта по дням за последний месяц в разрезе стран, из которых пользователи заходили на сайт
    ```
    SELECT `country`, `date`, count(*) FROM `hits` WHERE `datetime` > toDateTime(toStartOfMonth(now())) GROUP BY `country`, toDate(`datetime`) as `date` ORDER BY `country`, `date`
    ```

4. Запрос на удаление неактуальных данных, более одного года, по пользователям
    ```
    ALTER TABLE `hits` DELETE WHERE `datetime` < toDateTime(today() - 365)
    ```
