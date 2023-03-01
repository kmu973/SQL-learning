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
npm install --save \
node-fetch csv adm-zip gdal-async
```




