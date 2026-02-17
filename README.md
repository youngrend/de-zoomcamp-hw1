
# DE Zoomcamp 2026 — Homework 1

## Question 1 — Understanding Docker images

Command used:

docker run -it --entrypoint=bash python:3.13
pip --version

Output:
pip 25.3 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)

Answer:
25.3




## Question 2 — Docker networking and docker-compose

pgAdmin connects to Postgres via the Docker network, so it uses the container/service hostname and the container port (5432), not the host-mapped port (5433).

Answer: db:5432



## Question 3 — Counting short trips

```sql
SELECT COUNT(*) AS short_trips
FROM read_parquet('green_tripdata_2025-11.parquet')
WHERE lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
  AND trip_distance <= 1;
```
Answer: 8007


## Question 4 — Longest trip for each day

```sql
  SELECT DATE(lpep_pickup_datetime) AS pickup_day,
       MAX(trip_distance) AS max_trip_distance
FROM read_parquet('green_tripdata_2025-11.parquet')
WHERE lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
  AND trip_distance < 100
GROUP BY 1
ORDER BY max_trip_distance DESC
LIMIT 1;
```
Output: 2025-11-14 (88.03 miles)
Answer: 2025-11-14


## Question 5 — Biggest pickup zone
```sql
SELECT z."Zone" AS pickup_zone,
       SUM(g.total_amount) AS total_amount_sum
FROM read_parquet('green_tripdata_2025-11.parquet') g
JOIN read_csv_auto('taxi_zone_lookup.csv') z
  ON g."PULocationID" = z."LocationID"
WHERE g.lpep_pickup_datetime >= TIMESTAMP '2025-11-18'
  AND g.lpep_pickup_datetime <  TIMESTAMP '2025-11-19'
GROUP BY 1
ORDER BY total_amount_sum DESC
LIMIT 1;
```
Output: East Harlem North (9281.92)
Answer: East Harlem North


## Question 6 — Largest tip

```sql
WITH zones AS (
  SELECT * FROM read_csv_auto('taxi_zone_lookup.csv')
),
trips AS (
  SELECT *
  FROM read_parquet('green_tripdata_2025-11.parquet')
  WHERE lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
    AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
)
SELECT dz."Zone" AS dropoff_zone,
       MAX(tip_amount) AS max_tip
FROM trips t
JOIN zones pz ON t."PULocationID" = pz."LocationID"
JOIN zones dz ON t."DOLocationID" = dz."LocationID"
WHERE pz."Zone" = 'East Harlem North'
GROUP BY 1
ORDER BY max_tip DESC
LIMIT 1;
```

Output: Yorkville West (81.89)
Answer: Yorkville West