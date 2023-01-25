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
    - tool built on PostgreSQL platform

# ■ SQL-learning : PostgreSQL

## 1. manipulation [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-manipulation/cheatsheet)

1. **CREATE TABLE** creates a new table
("https://www.postgresql.org/docs/current/sql-createtable.html")
("https://www.postgresql.org/docs/current/sql-copy.html")

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

Quick Tabular data cleaning 
- csvkit : csvsql
```
csvsql \
  --db postgresql://postgres:postgres@localhost:5432/musa_509 \
  --tables csvkit_indego_stations \
  --create-if-not-exists \
  --insert \
  --overwrite \
  data/indego_stations.csv
```
- python Pandas
- R

3. **INSERT INTO** adds a new row to a table.
4. **SELECT** queries data from a table.
5. **ALTER TABLE** changes an existing table.
6. **UPDATE** edits a row in a table.
7. **DELETE FROM** deletes rows from a table.

## 2. query [cheatsheet](https://www.codecademy.com/learn/learn-sql/modules/learn-sql-queries/cheatsheet)

1. **SELECT** is the clause we use every time we want to query information from a database.
2. **AS** renames a column or table.
3. **DISTINCT** return unique values.
4. **WHERE** is a popular command that lets you filter the results of the query based on conditions that you specify.
5. **LIKE** and BETWEEN are special operators.
6. **AND** and **OR** combines multiple conditions.
7. **ORDER BY** sorts the result.
8. **LIMIT** specifies the maximum number of rows that the query will return.
9. **CASE** creates different outputs.


``

``
