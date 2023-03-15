```
instead of saving raw-data in local folder, let's extract and save data in cloud

1. open code 
2. create GC raw data and processed data buckets 
3. IAM (identity Access Management) 
4. install GC storage client libraries
5. Set up authentification
6. npm i dotenv / dotenv-cli (?)
7. extract data and put this into raw bucket
8. load raw data from bucket and process it into csv

```

# IAM
![image](https://user-images.githubusercontent.com/70645899/225352644-26b88bc9-cd9c-4ef9-8355-157aed58eac0.png)
![image](https://user-images.githubusercontent.com/70645899/225353450-f57ab602-564f-4051-bf9f-b7aa63d6a2a9.png)


# Google cloud storage client libraries
![image](https://user-images.githubusercontent.com/70645899/225355281-06f64722-f80f-4e6f-be88-2b021a30b2f1.png)
![image](https://user-images.githubusercontent.com/70645899/225355420-5ff7fbbb-a05b-43d6-bbb2-9e359187b44c.png)


# Set up authentification
**add keys to .gitignore**
![image](https://user-images.githubusercontent.com/70645899/225357210-0a29764f-498d-45b7-aa39-62210e744457.png)

# Code
```node
import fetch from 'node-fetch';
import storage from '@google-cloud/storage'

const client = new storage.Storage(
 {keyFilename: './keys/musa-coursework-8ce8259d04b6.json'}
);
const bucket = client.bucket('min_musa_509_raw_data')

const url = 'https://api.census.gov/data/2020/dec/pl?get=NAME,GEO_ID,P1_001N&for=block%20group:*&in=state:42%20county:*';
const resp = await fetch(url);
const data = await resp.json();

const jsonData = JSON.stringify(data);
const blob = bucket.file('census/census_population_2020.json');
await blob.save(jsonData, {resumable:false});
```

```
import * as csv from 'csv/sync';
import storage from '@google-cloud/storage'

const client = new storage.Storage(
  {keyFilename: './keys/musa-coursework-8ce8259d04b6.json'}
  );
const rawBucket = client.bucket('min_musa_509_raw_data')
const processedBucket = client.bucket('min_musa_509_processed_data')

const rawBlob = rawBucket.file('census/census_population_2020.json')
const [content] = await rawBlob.download();

const data = JSON.parse(content);

const outContent = csv.stringify(data);

const processedBlob = processedBucket.file('census_population_2020/data.csv');
await processedBlob.save(outContent, {resumable:false});

```
