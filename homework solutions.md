# Homework

## Question 0

```sql
WITH zones as
    (Select location_id, zone from taxi_zone)
Select
    tpep_dropoff_datetime, dolocationid, zones.zone
from
    trip_data
JOIN
    zones
ON
    zones.location_id = trip_data.dolocationid
ORDER BY
    tpep_dropoff_datetime desc LIMIT 10;
```

## Question 1

```sql
CREATE MATERIALIZED VIEW trip_times AS 
    WITH set as (
        SELECT
            pu_zone.zone as pu_zone, 
            do_zone.zone as do_zone, 
            (tpep_dropoff_datetime-tpep_pickup_datetime) as trip_time
        FROM trip_data
        JOIN taxi_zone as pu_zone
            ON trip_data.pulocationid = pu_zone.location_id
        JOIN taxi_zone as do_zone
            ON trip_data.dolocationid = do_zone.location_id
    )

    SELECT
        pu_zone, 
        do_zone,
        AVG(trip_time) as avg_trip_time,
        MIN(trip_time) as min_trip_time,
        MAX(trip_time) as max_trip_time
    FROM set
    GROUP BY
        pu_zone,
        do_zone
    ORDER BY avg_trip_time desc;
```

```sql
-- to get the top avg:
select 
    pu_zone || ' --> ' || do_zone,
    avg_trip_time 
from trip_times 
order by avg_trip_time desc 
limit 10;

---              ?column?                        | avg_trip_time 
-------------------------------------------------------+---------------
-- Yorkville East --> Steinway                           | 23:59:33
-- Stuy Town/Peter Cooper Village --> Murray Hill-Queens | 23:58:44
-- Washington Heights North --> Highbridge Park          | 23:58:40
-- ...etc.
```

## Question 2

```sql
CREATE MATERIALIZED VIEW trip_times_2 AS 
    WITH set as (
        SELECT
            pu_zone.zone as pu_zone, 
            do_zone.zone as do_zone, 
            (tpep_dropoff_datetime-tpep_pickup_datetime) as trip_time
        FROM trip_data
        JOIN taxi_zone as pu_zone
            ON trip_data.pulocationid = pu_zone.location_id
        JOIN taxi_zone as do_zone
            ON trip_data.dolocationid = do_zone.location_id
    )

    SELECT
        pu_zone, 
        do_zone,
        count(*) as cnt,
        AVG(trip_time) as avg_trip_time,
        MIN(trip_time) as min_trip_time,
        MAX(trip_time) as max_trip_time
    FROM set
    GROUP BY
        pu_zone,
        do_zone
    ORDER BY avg_trip_time desc;
```

```sql
select 
    pu_zone || ' --> ' || do_zone,
    cnt,
    avg_trip_time 
from trip_times_2
order by avg_trip_time desc 
limit 10;

---                       ?column?                        | cnt | avg_trip_time 
-------------------------------------------------------+-----+---------------
-- Yorkville East --> Steinway                           |   1 | 23:59:33
-- Stuy Town/Peter Cooper Village --> Murray Hill-Queens |   1 | 23:58:44
-- Washington Heights North --> Highbridge Park          |   1 | 23:58:40
-- ...etc....
```

## Question 3

```sql
CREATE MATERIALIZED VIEW busiest_zones AS 
    WITH lpt as (
        SELECT max(tpep_pickup_datetime) as latest_pickup_time
        FROM trip_data
    )

    SELECT
        pu_zone.zone as pu_zone,
        count(*) as pu_cnt
    FROM lpt,
        trip_data
    JOIN taxi_zone as pu_zone
        ON trip_data.pulocationid = pu_zone.location_id
    WHERE tpep_pickup_datetime >= (latest_pickup_time - INTERVAL '17 hours')
    GROUP BY pu_zone
    ORDER BY pu_cnt desc
    LIMIT 3;

--       pu_zone       | pu_cnt 
---------------------+--------
-- LaGuardia Airport   |     19
-- Lincoln Square East |     17
-- JFK Airport         |     17
--(3 rows)
```

```sql
-- test to check if returns 19 rows correctly for laguardia.
WITH lpt as (
        SELECT max(tpep_pickup_datetime) as latest_pickup_time
        FROM trip_data
    )

    SELECT
        pu_zone.zone as pu_zone,
        trip_data.tpep_pickup_datetime,
        lpt.latest_pickup_time
    FROM lpt,
        trip_data
    JOIN taxi_zone as pu_zone
        ON trip_data.pulocationid = pu_zone.location_id
    WHERE tpep_pickup_datetime >= (latest_pickup_time - INTERVAL '17 hours')
        AND pu_zone.zone = 'LaGuardia Airport'
    ORDER BY tpep_pickup_datetime desc
    LIMIT 21;
-- correct..
```