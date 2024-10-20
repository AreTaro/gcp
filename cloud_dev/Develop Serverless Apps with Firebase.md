# Develop Serverless Apps with Firebase

## Objective

- Create a frontend solution using a Rest API and Firestore database based on [pet-theory](https://github.com/rosera/pet-theory/tree/main) repository.

## Definitions

**REST API**: Representation State Transfer Application Programming Interface . Think of it like a waiter taking order to the kitchen and bringing back the plate. It defines a set of rules and conventions for how your applications can interact with each other over HTTP.

**Firestore**: NoSQL document database that is part of Firebase platform. 

**Firebase**: Platform developed by Google for building web and mobile applications.

**Node Package Manager (npm):** A vast online repository of JavaScript packages.

## Preliminary

Link the project:

```bash
gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')
```

_Break down_

1. `gcloud projects list --format='value(PROJECT_ID)`: This portion of the command lists all of your Google Cloud projects, filters them to include only those with 'qwiklabs-gcp' in their name or description, and extract the project ID of the first matching project. The `--format='value(PROJECT_ID) part ensures only the project ID is returned.
2. `gcloud config set project`: This portion of the command uses the project ID that was found in the first part to set the active project in gcloud configuration.


Clone the repo:
```bash
git clone https://github.com/rosera/pet-theory.git
```

Setup some environment variables for later use:

```bash
export REGION=[REGION]]
export SERVICE_NAME=netflix-dataset-service
export FRONTEND_STAGING_SERVICE_NAME=frontend-staging-service
export FRONTEND_PRODUCTION_SERVICE_NAME=frontend-production-service
```

## Create a Firestore database

There are two ways to create a Firestore database:
1. **Using the GCP console:** Navigate to the Firestore section and create a Database
2. **Using CLI:** Utilize the command-line interface to create the databasae

This guide will focus on the second approach, using the Firebase CLI.

### Enable the Cloud Run API. 
This is necessary to deploy and manage Cloud Run Services.

```bash
gcloud services enable run.googleapis.com
```

### Create a Firestore database

```bash
gcloud firestore databases create --location=$REGION
```

## Populate the database

### Navigate to the firebase-import-csv/solution directory.

```bash
cd pet-theory/lab06/firebase-import-csv/solution
```

### Install Node Package Manager (npm).

```bash
npm install
```

### Execute JavaScript code (index.js) to import CSV data into Firestore.

```bash
node index.js netflix_titles_original.csv
```
In essence, the index.js file is designed to:
1. Read data from a CSV file.
2. Parse taht data into individual records.
3. Connect to a Firestore database.
4. Write those records to the database using batches for efficiency.
5. Log any errors or successes that occur during the process to Google Cloud Logging.

## Create REST API

### Access to `pet-theory/lab06/firebase-rest-api/solution-01`

```bash
cd ~/pet-theory/lab06/firebase-rest-api/solution-01
```

### Install Node Package Manager. 

```bash
npm install
```
It's always a good practice to run "npm install" in each project directory to ensure that you have the correct dependencies installed for that specific project. This helps avoid unexpected errors and ensure consistent behavior.

### Build and Deploy the code to Google Container Registry

```bash
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1
```

The command `gcloud builds submit` is used to build a Docker image and push it to the Google Container Registry (GCR). It works by looking for `Dockerfile` in the current directory or a specified directory.

The `Docker file` is a text file that contains instructions for building a Docker image. It specifies the base image, dependencies, application code, and other configurations needed to create the image.

**Dockerfile**
```dockerfile
FROM node:12-slim
WORKDIR /usr/src/app
COPY package.json package*.json ./
RUN npm install --only=production
COPY . .
CMD [ "npm", "start" ]
```
For more details, please refer to the git repository.

### Deploy the image as a Cloud Run service.

```bash
gcloud beta run deploy $SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 --allow-unauthenticated --region=$REGION
```

This command deploys a container image as a new service to Cloud Run. `--image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1` specifies the Docker image to use for the service. `gcr.io` is the domain for Google Container Registry, where the image is stored, `rest-api` is the name of the image, `0.1` is the tag or version of the image. `--allow-unauthenticated` makes the service publicly accessible. Without it, users would need to be authentificated to access Cloud Run service.

To get the copy of the deployed service URL, two solutions:
1. **Use the console:** Go to Cloud Run in the GCP console, click the service name and copy the url from the service name.
2. **Use CLI:** Use the following command to get the URL of the deployed service: `gcloud run services describe $SERVICE_NAME --platform=managed --region=$REGION --format="value(status.url)"`

```bash
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --platform=managed --region=$REGION --format="value(status.url)")
```

### Test the REST API by making a GET request.

```bash
curl -X GET $SERVICE_URL
```

## Deploy an updated revision of the code to access the Firestore DB

Access to `pet-theory/lab06/firebase-rest-api/solution-02` and do the same as in `solution-01` with the container image `gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2`

```bash
cd ~/pet-theory/lab06/firebase-rest-api/solution-02
npm install                                      
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2  
gcloud beta run deploy $SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 --allow-unauthenticated --region=$REGION
```

Get the copy of the service URL and test the REST API by making a GET request.

```bash
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --platform=managed --region=$REGION --format="value(status.url)")
curl -X GET $SERVICE_URL/2019
```

## Deploy the Production Frontend
Update the Staging Frontend to use the Firestore database.
*Mettre à jour le Frontend (interface de l'utilisateur) de préproduction pour utiliser la base de données Firestore*

### Access `pet-theory/lab06/firebase-frontend/public`
```bash
cd ~/pet-theory/lab06/firebase-frontend/public
```

### Update the frontend application to use the REST API.
<!-- Suggested code may be subject to a license. Learn more: ~LicenseLog:1723497486. -->
```bash
sed -i 's/^const REST_API_SERVICE = "data\/netflix\.json"/\/\/ const REST_API_SERVICE = "data\/netflix.json"/' app.js
```
This commands finds the line in the app.js file that defines the `REST_API_SERVICE` constant with a value of `"data/netflix.json"` and commments out that line by adding `//` at the beginning. [It may have better solution]

*Breakdowm:*
- `sed`: This command stands for "stream editor". 
- `-i`: This flag indicates to make the change directly to the original file. 
- `s/old/new/`: This is the substitution command fromat for `sed`. It tells `sed` to replace the string "old" with the string "new".
- `app.js`: This is the target file where the changes will be applied.

### Append the year to the SERVICE_URL
```bash
sed -i "1i const REST_API_SERVICE = \"$SERVICE_URL/2020\"" app.js
 ```
This command inserts the following line of JavaScript code at the very beginning (line 1) of the `app.js` file:
```javascript
const REST_API_SERVICE = "[SERVICE_URL]/2020"
```
*Breakdown*:
- `1` specifies the line number where the insertion should occur.
- `i` signifies the "insert" command, telling `sed` to add the following text before the specified line.
- For other explanations, please refer to the previous paragraph.

### Use Cloud Build to tag and deploy image revision to Container Registry
```bash
npm install                                     
cd ~/pet-theory/lab06/firebase-frontend         
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1
gcloud beta run deploy $FRONTEND_STAGING_SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 --region=$REGION --quiet
```

## Deploy the production frontend
```bash
cd ~/pet-theory/lab06/firebase-frontend/public
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1 
gcloud beta run deploy $FRONTEND_PRODUCTION_SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1 --region=$REGION --quiet  # Deploy the production frontend to Cloud Run
```

## source
# Develop Serverless Apps with Firebase: Challenge Lab
# Deploy to Kubernetes in Google Cloud: Challenge Lab.md
# https://github.com/drpcc/Labs_solutions.git
# https://github.com/Aditya2086/Google-Cloud-Skills-Boost.git