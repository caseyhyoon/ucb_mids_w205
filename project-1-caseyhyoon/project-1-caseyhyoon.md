# Project 1

Author: Casey Yoon

### Part 1 - Querying Data with BigQuery

#### Initial Queries

- Question 1: What's the size of this dataset? (i.e., how many trips)
    
    * answer: 983648
    * SQL Query:
        ```sql
        #standardSQL
        SELECT count(*)
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        ```
        
- Question 2: What is the earliest start date and time and latest end date and time for a trip?

    * answer: 
        - earliest: 8/29/2013 9:08AM
        - latest: 8/31/2016 11:48PM
    * SQL Query:
        ```sql
        #standardSQL
        SELECT min(start_date), max(end_date)
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        ```
        
- Question 3: How many bikes are there?

    * answer: 700 bikes
    * SQL Query:
        ```sql
        #standardSQL
        SELECT count(distinct bike_number)
        FROM `bigquery-public-data.san_francisco.bikeshare_trips`
        ```
        
        
#### Questions of my own

- Question 1: Top 5 zip codes for customers? For subscribers?
    * answer: (Top 1 customers zip, Top 1 subscribers zip, etc...)
        1. nil (Lot of customers don't input zip codes), 94107
        2. (blank), 94105
        3. 94107, 94133
        4. 94105, 94103
        5. 94103, 94111
        
    * SQL Query: (ORDER BY num_subscribers or num_customers DESC)
        ```sql
        #standardSQL
        SELECT
            zip_code AS zip,
            count(IF(subscriber_type="Subscriber", subscriber_type, Null)) AS num_subscribers,
            count(IF(subscriber_type="Customer", subscriber_type, Null)) AS num_customers,
        FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
        GROUP BY zip, subscriber_type
        ```
        
- Question 2: What is the longest and shortest trip?
    * answer:
        - longest: 199 days
        - shortest: 1 minute
    * SQL Query:
        ```sql
        #standardSQL
        SELECT max(duration_sec/86400)
        FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`;

        SELECT min(duration_sec)
        FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
        ```
        
- Question 3: What is the average trip duration for subscribers vs customers?
    * Answer:
        - Subscribers: About 9-10 minutes across the day
        - Customers: 40+ minutes across the day
    * SQL Query:
    
```sql
SELECT avg(duration_sec/60) AS avg_duration_min,
    CASE 
        WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
        WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
        WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
        WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
        WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
        WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
        WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
        END AS start_hour_str
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE subscriber_type = 'Subscriber'
GROUP BY start_hour_str, subscriber_type
```

```sql
SELECT avg(duration_sec/60) AS avg_duration_min,
    CASE 
        WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
        WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
        WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
        WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
        WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
        WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
        WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
        END AS start_hour_str
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
WHERE subscriber_type = 'Customer'
GROUP BY start_hour_str, subscriber_type
```
### Part 2 - Querying data from the BigQuery CLI


##### Rerun Queries from Part 1

- Question 1: What's the size of this dataset? (i.e., how many trips)

```
! bq query --use_legacy_sql=FALSE '
    SELECT count(*)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```

- Question 2: What is the earliest start date and time and latest end date and time for a trip?

```
! bq query --use_legacy_sql=FALSE '
    SELECT min(start_date), max(end_date)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```

- Question 3: How many bikes are there?

```
! bq query --use_legacy_sql=FALSE '
    SELECT count(distinct bike_number)
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
```

##### New Query

- Question: How many trips are in the morning vs in the afternoon? (Assuming morning = 6AM-8AM, afternoon = 5PM-7PM)

    * Answer: 220,514 morning vs. 251,942 afternoon trips
    * SQL Query:
    
```
! bq query --use_legacy_sql=FALSE '
    SELECT COUNT(*), 
       CASE 
           WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
           WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
           WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
           END AS start_hour_str
    FROM `bigquery-public-data.san_francisco.bikeshare_trips`
    GROUP BY start_hour_str'
```

#### Project Questions

- Question 1: What are the most popular stations for subscribers?

- Question 2: What are the most popular stations for customers?

- Question 3: What are the most popular times for customer trips as opposed to subscriber trips?

- Question 4: What are the top 5 stations with the highest bike availabilities in the morning or afternoon?

- Question 5: What are the top 5 stations with the highest bike availabilities in mid day or early afternoon?

- Question 6: What are the top 5 most popular customer trips for the weekend and their average trip duration in minutes?

#### Answers

- Question 1: What are the most popular stations for subscribers in the morning or afternoon?
    * Answer:
        1. San Francisco Caltrain (Townsend at 4th)
        2. San Francisco Caltrain 2 (330 Townsend)
        3. Temporary Transbay Terminal (Howard at Beale)
        4. Harry Bridges Plaza (Ferry Building)
        5. Steuart at Market
    * SQL Query:
```
! bq query --use_legacy_sql=FALSE '
    SELECT COUNT(*) AS count, start_station_name
    FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
    WHERE subscriber_type = 'Subscriber' AND
  ((EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8) OR
  (EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19) )
    GROUP BY start_station_name, subscriber_type
    ORDER BY count DESC'
```


- Question 2: What are the most popular stations for customers?
    * Answer:
        Embarcadero at Sanome,
        Harry Bridges Plaza (Ferry Building),
        University and Emerson,
        Embarcadero at Vallejo
    * SQL Query:
    
```
! bq query --use_legacy_sql=FALSE '
    SELECT COUNT(*) AS count, start_station_name, end_station_name, subscriber_type
    FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
    WHERE subscriber_type = 'Customer'
    GROUP BY start_station_name, end_station_name, subscriber_type
    ORDER BY count DESC'
```

- Question 3: What are the most popular times for customer trips as opposed to subscriber trips?

    * Answer:
        - Customer: Early Afternoon, Mid Day
        - Subscribers: Afternoon, Morning
    * SQL Query:
```
! bq query --use_legacy_sql=FALSE '
    SELECT COUNT(*) AS count,
       CASE 
           WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
           WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
           WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
           END AS start_hour_str
    FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
    WHERE subscriber_type="Customer"
    GROUP BY start_hour_str'
```

- Question 4: What are the top 5 stations with the highest bike availabilities in the morning or afternoon?
    * Answer:
        1. 5th St at Folsom St
        2. Harry Bridges Plaza (Ferry Building) 
        3. San Francisco Caltrain 2 (330 Townsend) 
        4. San Jose Diridon Caltrain Station
        5. 2nd at Townsend
    * SQL Query:
```
! bq query --use_legacy_sql=FALSE '
    SELECT stations.name, avg(status.bikes_available) AS bike_availability
FROM `my-first-project-287722.bike_trip_data.bikeshare_status` status
LEFT JOIN `my-first-project-287722.bike_trip_data.bikeshare_stations` stations
  ON status.station_id = stations.station_id AND
  ((EXTRACT(HOUR FROM status.time) >= 6 AND EXTRACT(HOUR FROM status.time) <= 8) OR
  (EXTRACT(HOUR FROM status.time) >= 17 AND EXTRACT(HOUR FROM status.time) <= 19))
GROUP BY stations.name
ORDER BY bike_availability DESC'
```

- Question 5: What are the top 5 stations with the highest bike availabilities in mid day or early afternoon?
    * Answer:
        1. Market at Sansome
        2. 2nd at Townsend
        3. 5th at Folsom St
        4. San Jose Diridon Caltrain Station
        5. Redwood City Caltrain Station
    * SQL Query:
```
! bq query --use_legacy_sql=FALSE '
    SELECT stations.name, avg(status.bikes_available) AS bike_availability
    FROM `my-first-project-287722.bike_trip_data.bikeshare_status` status
    LEFT JOIN `my-first-project-287722.bike_trip_data.bikeshare_stations` stations
      ON status.station_id = stations.station_id AND
      ((EXTRACT(HOUR FROM status.time) >= 11 AND EXTRACT(HOUR FROM status.time) <= 13) OR
      (EXTRACT(HOUR FROM status.time) >= 14 AND EXTRACT(HOUR FROM status.time) <= 16))
    GROUP BY stations.name
    ORDER BY bike_availability DESC'
```

- Question 6: What are the top 5 most popular customer trips for the weekend and their average trip duration in minutes?

```
! bq query --use_legacy_sql=FALSE '
SELECT COUNT(*) AS count,
       avg(duration_sec/60) AS avg_min,
       start_station_name,
       CASE 
           WHEN EXTRACT(HOUR FROM start_date) <= 5  OR EXTRACT(HOUR FROM start_date) >= 23 THEN "Nightime"
           WHEN EXTRACT(HOUR FROM start_date) >= 6 and EXTRACT(HOUR FROM start_date) <= 8 THEN "Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 9 and EXTRACT(HOUR FROM start_date) <= 10 THEN "Mid Morning"
           WHEN EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13 THEN "Mid Day"
           WHEN EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16 THEN "Early Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 17 and EXTRACT(HOUR FROM start_date) <= 19 THEN "Afternoon"
           WHEN EXTRACT(HOUR FROM start_date) >= 20 and EXTRACT(HOUR FROM start_date) <= 22 THEN "Evening"
           END AS start_hour_str
FROM `my-first-project-287722.bike_trip_data.bikeshare_trips`
WHERE EXTRACT(DAYOFWEEK FROM start_date) IN (1, 7) AND
      ((EXTRACT(HOUR FROM start_date) >= 11 and EXTRACT(HOUR FROM start_date) <= 13) OR
      (EXTRACT(HOUR FROM start_date) >= 14 and EXTRACT(HOUR FROM start_date) <= 16))
GROUP BY start_station_name, start_hour_str
ORDER BY count DESC'
```