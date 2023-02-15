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

```python

import json

# Load the data from the GeoJSON file
with open('opa_properties.geojson') as f:
    data = json.load(f)

# Write the data to a JSONL file
with open('opa_properties.jsonl', 'w') as f:
    for feature in data['features']:
        row = feature['properties']
        row['geog'] = json.dumps(feature['geometry'])
        f.write(json.dumps(row) + '\n')
        
```
