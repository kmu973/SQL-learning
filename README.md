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
## 3. AGGREGATE [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-aggregate-functions/cheatsheet)

1. **COUNT()**: count the number of rows
2. **SUM()**: the sum of the values in a column
3. **MAX()/MIN()**: the largest/smallest value
4. **AVG()**: the average of the values in a column
5. **ROUND()**: round the values in the column
6. **GROUP BY** is a clause used with aggregate functions to combine data from one or more columns.
7. **HAVING** limit the results of a query based on an aggregate property.

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

## 2. Loading Spatial data Files

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
- **R**
