# ■ Database

## 1. NoSQL

 ### 1. Key-Value 
    - like a dictionary
    - much faster than normal SQL data, using user profile dataset and caching it
    - Redis, Memcahced
    
 ### 2. Document
    - semi-structured documents 
    - dictionary but having you can search verb, noun, adj...
    - MongoDB
    
 ### 3. Graph
    - Neo4j, TAO
    - optimized to query relations btw data
    
## 2. SQL

 ### 1. Relational 
    - MySQL, **PostgreSQL**
    - Not meaning relational btw data
    
# ■ PostgreSQL
    - Server hosting database
    - Client : PGAdim, Postico, QGIS, ogr2ogr

# ■ PostGIS
- https://postgis.net/workshops/postgis-intro/
- https://postgis.net/docs/reference.html#Geometry_Processing
- tool built on PostgreSQL platform


# ■ SQL-learning

## 1. MANIPULATION [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-manipulation/cheatsheet)

1. **CREATE TABLE** creates a new table
3. **INSERT INTO** adds a new row to a table.
4. **SELECT** queries data from a table.
5. **ALTER TABLE** changes an existing table.
6. **UPDATE** edits a row in a table.
7. **DELETE FROM** deletes rows from a table.

```
-- CREATE TABLE
CREATE TABLE friends (
  id INTEGER,
  name TEXT,
  birthday DATE
);

-- INSERT INFO
INSERT INTO friends
VALUES (1,'PARK', '1992-07-08');

INSERT INTO friends
VALUES (2,'MIN', '1991-05-20');

INSERT INTO friends
VALUES (3,'BIDEN', '1942-11-20');

-- UPDATE INFO
UPDATE friends
SET name = "SON"
WHERE id = 1;

-- ADD COLUMN
ALTER TABLE friends
ADD COLUMN email TEXT;

UPDATE friends
SET email = "unkown@gmail.com"
WHERE id = 1;

DELETE FROM friends
WHERE id = 2;

SELECT * FROM friends;


```

## 2. QUERY [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-queries/cheatsheet)

1. **SELECT** is the clause we use every time we want to query information from a database.
2. **AS** renames a column or table.
3. **DISTINCT** return unique values.
4. **WHERE** is a popular command that lets you filter the results of the query based on conditions that you specify.
5. **LIKE** and BETWEEN are special operators.
6. **AND** and **OR** combines multiple conditions.
7. **ORDER BY** sorts the result.
8. **LIMIT** specifies the maximum number of rows that the query will return.
9. **CASE** creates different outputs.

```
SELECT * FROM nomnom;

SELECT DISTINCT neighborhood
FROM nomnom;

SELECT DISTINCT cuisine
FROM nomnom;

SELECT *
FROM nomnom
WHERE cuisine = 'Chinese';

SELECT *
FROM nomnom
WHERE review > 4;

SELECT *
FROM nomnom
WHERE cuisine = 'Italian'
AND price = '$$$';

SELECT *
FROM nomnom
WHERE name LIKE '%meatball%';

SELECT *
FROM nomnom
WHERE neighborhood = 'Midtown'
OR neighborhood = 'Downtown'
OR neighborhood = 'Chinatown';

SELECT *
FROM nomnom
WHERE health IS NULL;

SELECT *
FROM nomnom
ORDER BY review DESC
LIMIT 10;

SELECT name,
  CASE
    WHEN review > 4.5 THEN "Extraordinary"
    WHEN review > 4 THEN "Excellent"
    WHEN review > 3 THEN "Good"
    WHEN review > 2 THEN "Fair"
    ELSE "Poor"
  END AS 'Review'
FROM nomnom;

```

**Qeury Spatial data**
```
SELECT * FROM indego_station_statuses
WHERE ST_Y(the_geog::geometry) > 39.95221848137947
ORDER BY ST_Y(the_geog::geometry);
```


## 3. AGGREGATE [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-aggregate-functions/cheatsheet)

1. **COUNT()**: count the number of rows
2. **SUM()**: the sum of the values in a column
3. **MAX()/MIN()**: the largest/smallest value
4. **AVG()**: the average of the values in a column
5. **ROUND()**: round the values in the column
6. **GROUP BY** is a clause used with aggregate functions to combine data from one or more columns.
7. **HAVING** limit the results of a query based on an aggregate property.


## 4. MULTIPLE [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-multiple-tables/cheatsheet)

1. **JOIN** will combine rows from different tables if the join condition is true.
2. **LEFT JOIN** will return every row in the left table, and if the join condition is not met, NULL values are used to fill in the columns from the right table.
3. **Primary key** is a column that serves a unique identifier for the rows in the table.
4. **Foreign key** is a column that contains the primary key to another table.
5. **CROSS JOIN** lets us combine all rows of one table with all rows of another table.
6. **UNION** stacks one dataset on top of another.
7. **WITH** allows us to define one or more temporary tables that can be used in the final query.


# ■ PostgreSQL

## 1. Loading Tabular data Files
(https://www.postgresql.org/docs/current/sql-createtable.html)
(https://www.postgresql.org/docs/current/sql-copy.html)

```
DROP TABLE IF EXISTS indego_stations;

CREATE TABLE indego_stations
(
  id           INTEGER,
  name         TEXT,
  go_live_date DATE,
  status       TEXT
);

COPY indego_stations
FROM '...path to file goes here...'
WITH (FORMAT csv, HEADER true);

```

## 2. Quick Tabular data cleaning 

- **csvkit : csvsql**
```
csvsql \
  --db postgresql://postgres:postgres@localhost:5432/musa_509 \
  --tables csvkit_indego_stations \
  --create-if-not-exists \
  --insert \
  --overwrite \
  data/indego_stations.csv
```
- **python Pandas**
```
import pandas as pd
df = pd.read_csv('data/indego_stations.csv')
USERNAME = 'postgres'
PASSWORD = 'postgres'
DATABASE = 'musa_509'
df.to_sql(
    'indego_stations',
    f'postgresql://{USERNAME}:{PASSWORD}@localhost:5432/{DATABASE}',
    if_exists='replace',
    index=False,
)
```
- **R**
```
library(RPostgreSQL)

df <- read.csv('data/indego_stations.csv')

drv <- dbDriver('PostgreSQL')
con <- dbConnect(drv,
    user = 'postgres', password = 'postgres',
    dbname = 'musa_509', host = 'localhost')

if (dbExistsTable(con, 'r_indego_stations'))
    dbRemoveTable(con, 'r_indego_stations')

dbWriteTable(con,
    name = 'r_indego_stations',
    value = df, row.names = FALSE)
```

## 3. Loading Spatial data Files

- **ogr2ogr**
(https://gdal.org/programs/ogr2ogr.html)
(https://gdal.org/drivers/vector/pg.html#vector-pg)
(https://morphocode.com/using-ogr2ogr-convert-data-formats-geojson-postgis-esri-geodatabase-shapefiles/)
"driver" determine the types of formats
```
ogr2ogr \
  -f "PostgreSQL" \
  -nln "indego_station_statuses" \
  -lco "OVERWRITE=yes" \
  -lco "GEOM_TYPE=geography" \
  -lco "GEOMETRY_NAME=the_geog" \
  PG:"host=localhost port=5432 dbname=musa_509 user=postgres password=postgres" \
  "data/indego_station_statuses.geojson"
```

- **QGIS**
(https://www.crunchydata.com/blog/loading-data-into-postgis-an-overview)
QGIS don't like nested array(?)

- **python Pandas**
```
import geopandas as gpd
import sqlalchemy as sqa

# Load the shp into a DataFrame.
df = gpd.read_file('data/indego_station_statuses.geojson')

# Load the DataFrame into the database.
USERNAME = 'postgres'
PASSWORD = 'postgres'
DATABASE = 'musa_509'

engine = sqa.create_engine(
    f'postgresql://{USERNAME}:{PASSWORD}@localhost:5432/{DATABASE}'
)
df.to_postgis(
    'pandas_indego_station_statuses',
    engine,
    if_exists='replace',
    index=False,
)
```
---

# Data types, Aggregations, Geospatial Operations, and Joins

---

## PostgreSQL Data Types, a sampling

| Data Type | Example |
|---|---|
| Boolean | `true`, `false`, `null` |
| Numbers | `1`, `3.1415`, `-1.2e-7`, `null` |
| Text | `‘Musa’`, `‘a’`, `‘1234’`, `null` |
| Date/time (timestamp/time with/without time zone, date, interval) | `‘2020-09-08’::date` <br> `to_timestamp(1599572175)` <br> `Interval ‘4 days 3 hours 13 seconds’` |
| Money | `$10`, `¥10`, `€10`, `null` |
| Arrays of any type | `[1, 2, 3, 4]`, `[‘banana’, ‘orange’, ‘apple’]`, `null` |
| JSON (as JSONB) | `{‘a’: 1, ‘b’: 2, …}`, `null` |

- Boolean - `AND`, `OR`
- Numbers (integers, floating point numbers) - `+`, `-`, `AVG`, `SUM`, `CORR`
- Text - `STRING_AGG`, `||` (concatenate), `LIKE`, substrings, etc.
- Date/time - `-` (intervals), `EXTRACT` date parts, timezone conversion
  ```
  select extract (year from '2021-12-04'::date)
  select extract (month from '2021-12-04'::date)
  select extract (quarter from '2021-12-04'::date)  
  ```
---

## cast Convert a value from one data type to another

```
SELECT CAST('121' AS integer);
SELECT CAST(121 AS text);
SELECT CAST('SRID=4326;POINT(-75.16 39.95)' AS geometry);
```
```
PostgreSQL specific syntax
SELECT '121'::integer;
SELECT 121::text;
SELECT 'SRID=4326;POINT(-75.16 39.95)'::geometry;
```
---

## Working with `date` data

- Some of you may have run into problems reading the date columns in last week's exercises
- Dates are ambiguous (e.g. `6/10/2016`)
- It helps to be explicit when casting from `text` to `date`

**Converting `text` to `date`**

```sql
SELECT '6/10/2016'::date;

Depending on what language/region your computer is configured for, this may:
* Give the date value June 10, 2016
* Give the date value October 6, 2016
* Result in an error

SHOW DateStyle;

My setting is `ISO,MDY`. That's not right or wrong, it's just what's expected in a computer set to a US/English region and language.
`ISO` refers to [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) -- i.e. **`YYYY-MM-DD`**. Pretty much any DB will accept this. The other value (e.g. `MDY`) is specific to the "locale" (the region and language).

SELECT to_date('6/10/2016', 'DD/MM/YYYY');

ALTER TABLE my_table
ALTER COLUMN my_date TYPE date
    USING to_date(my_date, 'DD/MM/YYYY');

```
---

## Spatial data types

The PostGIS extension to PostgreSQL adds two data types:
- `geometry` : fast
- `geography` : slow

**The PostGIS documentation [recommends](http://postgis.net/workshops/postgis-intro/geography.html#why-not-use-geography):**
- If your data is geographically compact, use the `geometry` type with an [appropriate projection](http://epsg.io).
- If you need to measure distance with a dataset that is geographically dispersed, use the `geography` type.

<div style="text-align: center">

<div style="font-size: 2em">
**Use `geography`**



