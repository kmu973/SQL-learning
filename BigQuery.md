---

## Spatial Cloud PPT 
https://docs.google.com/presentation/d/1WcuSZ3BJz5wPfrbS9qNCJ20Axr6m0PIKLFMu-ljfdoI/edit#slide=id.p

#### Uses of the cloud in the world of spatial data
- Tile servers (quickly serving pre-rendered data)
- Large data repositories (Census data and APIs, etc.)
- Geospatial services (geocoders, routers, isochrones)

#### Google Cloud Platform

Cloud infrastructure providers generally bill on three dimensions:
- Storage -- how much data do you have stored, and where
- Compute -- how much time are your processes using to run
- Network -- how much data are you passing between services

----
## BigQuery

- Transaction Data base: real-time, immediate use
- Data Warehouse: waiting a bit, analytical , machine learning

#### Google BigQuery

- BigQuery ML (https://cloud.google.com/bigquery-ml/docs/reference/standard-sql/bigqueryml-syntax-e2e-journey)
- Public Datasets (https://cloud.google.com/bigquery/public-data, https://console.cloud.google.com/marketplace/browse?filter=solution-type:dataset)
- BigQuery GIS (https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions, https://cloud.google.com/bigquery/docs/geospatial-visualize#geo_viz_limitations)

#### Business intelligence

- Redash, Metabase, Apache Superset, Tableau
- Geospatial functions are very limited
- Carto

#### BigQuery SQL Dialect

- Notice the backtick (`) surrounding table references.
- Examples
![image](https://user-images.githubusercontent.com/70645899/219097657-3de9b501-51d5-4ca6-a716-4c832d47b496.png)
```
`bigquery-public-data.nasa_wildfire.past_week`
`bigquery-public-data.covid19_open_data.compatibility_view`
`musa-509.week-5-data.my_private_table`
```

----

## BigQuery 

#### 1. Download the OPA Properties dataset (GeoJson)
(https://opendataphilly.org/dataset/opa-property-assessments)

#### 2. Convert the OPA Properties dataset to JSON-L
GeoJson to JSON file

- python
```python

import json

# Load the data from the GeoJSON file
with open('opa_properties.geojson', encoding='utf-8') as f:
    data = json.load(f)

# Write the data to a JSONL file
with open('opa_properties.jsonl', 'w') as f:
    for feature in data['features']:
        row = feature['properties']
        row['geog'] = json.dumps(feature['geometry'])
        f.write(json.dumps(row) + '\n')
        
```
- javascript
```javascript
import fs from 'node:fs';

// Load the data from the GeoJSON file
const data = JSON.parse(
  fs.readFileSync('opa_properties.geojson'));

// Write the data to a JSONL file
const f = fs.createWriteStream('opa_properties.jsonl');
for (const feature of data.features) {
  const row = feature.properties;
  row.geog = JSON.stringify(feature.geometry);
  f.write(JSON.stringify(row) + '\n');
}
```

#### 3. Upload the resulting JSONL file to Google Cloud Storage

- Log in to the **Google Cloud Console** (https://console.cloud.google.com)
- Find the **Google Cloud Storage** service
- Create a new bucket in Google Cloud Storage (maybe call it `data_lake`)
- Create a new folder in the bucket (maybe call it `phl_opa_properties`)
- Upload the JSONL file to the folder

#### 4. Create a new dataset in BigQuery

- Navigare to the **BigQuery** service
- Create a new dataset in BigQuery (maybe call it `data_lake`)
- Also create a new dataset in Carto (maybe call it `phl`; we'll use this later)

#### 5. Create the external table

https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#create_external_table_statement

```sql
CREATE OR REPLACE EXTERNAL TABLE `min_data_lake.phl_opa_properties`
OPTIONS (
  description = 'Philadelphia OPA Properties - Raw Data',
  uris = ['gs://min_data_lake/phl_opa_properties/*.jsonl'],
  format = 'JSON',
  max_bad_records = 0
)

--- 

SELECT * FROM `min_data_lake.phl_opa_properties`
limit 100
```

#### 6. Create a native table from the external table

```sql
CREATE OR REPLACE TABLE `phl.opa_properties`
CLUSTER BY (geog)
AS (
  SELECT * EXCLUDE (geog), ST_GEOFROMGEOJSON(geog) AS geog
  FROM `data_lake.phl_opa_properties`
)
```
