# Chapter 02

# Load data into DuckDB 


```sql
CREATE OR REPLACE TABLE foods (
  food_name VARCHAR PRIMARY KEY, 
  color VARCHAR,
  calories INT, 
  is_healthy BOOLEAN
);

COPY foods FROM 'foods_no_heading.csv';

SELECT * 
FROM foods;
```

## COPY file options during load

```sql
COPY foods (food_name, is_healthy, color, calories)  
FROM 'foods_with_heading.tsv' (DELIM '\t', HEADER);
```


## Error handling with the COPY Statement

```sql
COPY foods FROM 'foods_error.csv';

COPY foods FROM 'foods_error.csv' (IGNORE_ERRORS true);
```


## File loading with read_csv

```sql
SELECT * 
FROM read_csv('food_prices.csv');

SELECT Delimiter, HasHeader, Columns
FROM sniff_csv('food_prices.csv');

DESCRIBE SELECT * 
FROM read_csv('food_prices.csv');
```

## Table creation with read_csv

```sql
CREATE OR REPLACE TABLE low_cost_foods 
AS
SELECT * 
FROM read_csv('food_prices.csv') 
WHERE price < 4.00;

SELECT * 
FROM low_cost_foods;
```

## Date formats 

```sql
SELECT * 
FROM read_csv('food_orders.csv');

SELECT *
FROM read_csv(
'food_orders.csv',
dateformat='%Y%m%d',
types={'order_date': 'DATE'}
);

SELECT * 
FROM read_csv('food_orders.csv'
, dateformat='%Y%m%d'
, columns={
   'food_name': 'VARCHAR', 
   'order_date': 'DATE', 
   'quantity': 'INTEGER'}
);
```

## Loading multiple files

```sql
SELECT food_name, color, filename   
FROM read_csv('food_collection/pizza*.csv', 
filename=true);
```

## Mixed schemas

```sql
SELECT *  
FROM read_csv('food_collection/*.csv');

SELECT *  
FROM read_csv('food_collection/*.csv', 
union_by_name=true);
```


# JSON Files

```sql
SELECT *  
FROM read_json('pizza_orders_records.json',  
format='newline_delimited');

SELECT *
FROM read_json('pizza_orders_array_of_records.json',
format='array');
```

# Parquet


## Loading Parquet Files

```sql
SELECT name, type, converted_type
FROM parquet_schema('food_orders.parquet');


SELECT *
FROM read_parquet('food_orders.parquet');

DESCRIBE
SELECT *
FROM read_parquet('food_orders.parquet');
```


# Exploring public data sets

- Visit  [Melbourne Bike Share Station Readings](https://data.melbourne.vic.gov.au/explore/dataset/melbourne-bike-share-station-readings-2011-2017/information/)
- Download the compressed CSV file locally to your local machine.
- Unzip the downloaded file to extract the files from the compressed file and save them on your computer
- You should have a file called `74id-aqj9.csv` 

## Loading the bike station readings

```sql

SELECT *
FROM read_csv('74id-aqj9.csv')
LIMIT 5;

DESCRIBE
SELECT *
FROM read_csv('74id-aqj9.csv');

.mode line

SELECT *
FROM read_csv('74id-aqj9.csv')
LIMIT 1;

.mode duckbox

CREATE OR REPLACE TABLE bikes AS 
SELECT * 
FROM read_csv( 
    '74id-aqj9.csv', 
    timestampformat='%Y%m%d%H%M%S', 
    types={'rundate': TIMESTAMP} 
); 


SELECT count(*)
FROM bikes;

SUMMARIZE 
SELECT *  
FROM bikes;

SELECT * EXCLUDE(count, q25, q50, q75) 
FROM (SUMMARIZE bikes); 
```

# Exporting table data

```sql
CREATE OR REPLACE TABLE bike_readings_april 
as 
SELECT * 
FROM bikes 
WHERE rundate between '2017-04-01' and '2017-04-30';

COPY bike_readings_april TO 'bike_readings_april.csv'; 

COPY bike_readings_april TO 'bike_readings_april.tsv' 
    (HEADER false, DELIM '\t');


COPY ( 
    SELECT NAME, RUNDATE 
    FROM bike_readings_april 
) TO 'bike_readings_april.json' 
    (FORMAT JSON, TIMESTAMPFORMAT '%d %B %Y'); 

COPY (
    SELECT *, date_trunc('day', rundate) AS rundate_truncated
    FROM bike_readings_april 
) TO 'bike_readings' 
(FORMAT PARQUET, PARTITION_BY (rundate_truncated), OVERWRITE_OR_IGNORE true);
```

