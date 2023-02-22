# Data Pipelines
---

## 1. Extracting (≒ Downloading)
  - Downloading data over HTTP
  - Working with geospatial file formats
  - Working with other file formats (like zip)
 
 #### python
 ```bash
mkdir -p ~/code/week06
cd ~/code/week06
touch extract_business_licenses.py

python3 -m venv env
source env/bin/activate
```

 #### node.js

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
