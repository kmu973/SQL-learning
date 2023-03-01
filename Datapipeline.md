# Data Pipelines
---

## 1-1. Extracting (≒ Downloading)
  - Downloading data over HTTP
  - Working with geospatial file formats
  - Working with other file formats (like zip)
 
 ### python
 ```bash
mkdir -p ~/code/week06
cd ~/code/week06
touch extract_business_licenses.py

python3 -m venv env
source env/bin/activate
env\Sourcs\Activate
```

 ### node.js

```bash
mkdir -p ~/code/week06
cd ~/code/week06

npm init -y
```


```javascript

import https from 'https';
import fs from 'fs';

const url = "https://phl.carto.com/api/v2/sql?q=SELECT+*+FROM+business_licenses&filename=business_licenses&format=geojson&skipfields=cartodb_id"
const filename = "business_licenses.geojson";

https.get(url, (response) => {
    const f = fs.createWriteStream(filename);
    response.pipe(f);
});


```

---

## 1-2. More Extracting (Et of EtLT)

### Extract > transform

- Downlaod data from web and convert to a format that can be loaded into BigQuery (JSON-L, CSV)

### files / streams / buffers

-python: file-like object: csv.reader, json.load, zipfile.ZipFile
-Node.js: fs.createReadStream, fs.createWriteStream

### Common libraries

-python: urllib.urlopen, requests, csv, zipfile, fiona&pyproj (built on GDAL)
```python
pip install requests fiona pyproj
pip freeze > requirements.txt
```
-Node.js: https.get, node-fetch, csv, adm-zip, gdal-async
```javascript
npm install --save node-fetch csv adm-zip gdal-async
```

### Extract census

- census API >> EXTRACT >> [raw data] >> TRANSFORM >> [prepared data]
- census API : "https://api.census.gov/data/2020/dec/pl?get=NAME,GEO_ID,P1_001N&for=block%20group:*&in=state:42%20county:*"


#### Import JSON
![image](https://user-images.githubusercontent.com/70645899/222208717-30f1a823-da02-4b0a-97e4-67c45665910d.png)


```javascript

// Step 1: Extract from url
import fetch from 'node-fetch'; 

// Step 3: Save JSON data to Step 2 folder
import fs from 'fs';
import { fileURLToPath } from 'url';

// Step 1 : Extract 
const url = "https://api.census.gov/data/2020/dec/pl?get=NAME,GEO_ID,P1_001N&for=block%20group:*&in=state:42%20county:*";
const resp = await fetch(url);
const data = await resp.json(); // Go into url, you can see it's JSON format

// Step 2 : Create folder to save Raw data
const __dirname = fileURLToPath(new URL('.', import.meta.url)); 
const RAW_DATA_DIR = `${__dirname}/raw_data`; 

// Step 3
const jsonData = JSON.stringify(data);
fs.writeFileSync(`${RAW_DATA_DIR}/census_population_2020.json`, jsonData);

```
#### convert into csv

![image](https://user-images.githubusercontent.com/70645899/222220495-5f8d6a55-5e56-43a1-98ac-9ecbe40d1dad.png)

```javascript

import fs from "fs/promises";
import { stringify } from "csv";
import { fileURLToPath } from 'url'; 

// Step 4 : Create folder to save Processed data
const __dirname = fileURLToPath(new URL('.', import.meta.url)); 
const RAW_DATA_DIR = `${__dirname}/raw_data`; 
const PROCESSED_DATA_DIR = `${__dirname}/processed_data`; 

// Step 5 : Read JSON file to transform into csv
const content = await fs.readFile(`${RAW_DATA_DIR}/census_population_2020.json`, { encoding: 'utf8' });
const data = JSON.parse(content);

// Step 6 : convert JSON data into csv and save it
const outContent = stringify(data);
await fs.writeFile(`${PROCESSED_DATA_DIR}/census_population_2020.csv`, outContent, { encoding: 'utf8' });


```

