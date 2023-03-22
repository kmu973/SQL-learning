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
9. turn local scripts (extract_census, prepare_census) into google cloud function 

```

# IAM

<img src="https://user-images.githubusercontent.com/70645899/225352644-26b88bc9-cd9c-4ef9-8355-157aed58eac0.png" width="500">
<img src="https://user-images.githubusercontent.com/70645899/225353450-f57ab602-564f-4051-bf9f-b7aa63d6a2a9.png" width="500">



# Google cloud storage client libraries

<img src="https://user-images.githubusercontent.com/70645899/225355281-06f64722-f80f-4e6f-be88-2b021a30b2f1.png" width="500">
<img src="https://user-images.githubusercontent.com/70645899/225355420-5ff7fbbb-a05b-43d6-bbb2-9e359187b44c.png" width="500">


# Set up authentification
**add keys to .gitignore**
<img src="https://user-images.githubusercontent.com/70645899/225357210-0a29764f-498d-45b7-aa39-62210e744457.png" width="200">


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

```node
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

# Google cloud function

https://github.com/GoogleCloudPlatform/functions-framework-nodejs

```node
npm install @google-cloud/functions-framework --save
```
<img src="https://user-images.githubusercontent.com/70645899/225385718-6bb4f90b-6cf7-47d7-ba68-44e123547f06.png" width="500">
<img src="https://user-images.githubusercontent.com/70645899/225388760-9ba2b3b7-4f15-49f1-b6da-650276adf523.png" width="500">

```node 
import fetch from 'node-fetch';
import storage from '@google-cloud/storage';
import functions from '@google-cloud/functions-framework';



functions.http('extract_data', async (req, res) => {
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
 
 res.status(200).send('OK')

});

```

-------

# Deploying cloud

<img src="https://user-images.githubusercontent.com/70645899/226938025-63bfad09-8a83-4f83-a9cc-b61f7388e75d.png" width="300">

- estimate cost (https://cloud.google.com/products/calculator)
- free tier (https://cloud.google.com/free/docs/free-cloud-features)
- billing setting (https://console.cloud.google.com/billing)

```gcloud

gcloud functions deploy extract-census `
  --region us-central1 `
  --runtime nodejs18 `
  --trigger-http `
  --source src/extract_census `
  --entry-point extract_data `
  --service-account <service-account>

```

-test function https://console.cloud.google.com/functions/list

# Create workflow

- https://console.cloud.google.com/workflows
- https://cloud.google.com/workflows/docs/reference/stdlib/overview
- trigger url

```
main:
    steps:
    - extract:
        call: http.post
        args:
            url: https://us-central1-musa-coursework.cloudfunctions.net/extract-census
            auth: 
                type: OIDC
    - prepare:
        call: http.post
        args:
            url: https://us-central1-musa-coursework.cloudfunctions.net/extract-census
            auth:
                tyep: OIDC
```
Settup service account
```
gcloud projects add-iam-policy-binding \
  <PROJECT_ID> \
  --member serviceAccount:<EMAIL> \
  --role roles/cloudfunctions.invoker

>>vscode powershell
gcloud projects add-iam-policy-binding "musa-coursework" --member "serviceAccount:data-pipeline-user@musa-coursework.iam.gserviceaccount.com" --role "roles/cloudfunctions.invoker"

```
