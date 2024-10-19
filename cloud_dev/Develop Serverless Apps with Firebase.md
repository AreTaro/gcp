# Develp Serverless Apps with Fire base: Challenge lab

## Challenge

- Create a frontedn solution using a Rest API and Firestore database

## Definitions

**Rest API**: Representation State Transfer Application Programming Interface. Think of it like a waiter taking order to the kitchnen and bringing back the plate. It defines a set of rules and conventions for how your applications can interact with each other over HTTP.

**Firestore**: NoSQL document database that is part of Firebase platform. 

**Firebase**: platform developed by Google for building web and mobile applications.

## Preliminary

Link the project:

```yaml
gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')
```
Clone the repo:
```yaml
git clone https://github.com/rosera/pet-theory.git
```

## Task1: Create a Firestore database

There are two ways to create a Firestore database:
1. **Using the GCP console:** Navigate to the Firestore section and create a Database
2. **Using CLI:** Utilize the command-line interface to create the databasae

This guide will focus on the second approach, using the Firebase CLI.

First setup some environment variables for later use:

```yaml
export REGION=[REGION]]
export SERVICE_NAME=netflix-dataset-service
export FRONTEND_STAGING_SERVICE_NAME=frontend-staging-service
export FRONTEND_PRODUCTION_SERVICE_NAME=frontend-production-service
```

Enable the Cloud Run API. This is necessary to deploy and manage Cloud Run Services (necessary?)

```yaml
gcloud services enable run.googleapis.com
```

Create a Firestore database

```yaml
gcloud firestore databases create --location=$REGION
```

# Navigate to the firebase-import-csv/solution directory.
cd pet-theory/lab06/firebase-import-csv/solution

# Install the project dependencies.
npm install

# Run the index.js script with the netflix_titles_original.csv file as an argument.
node index.js netflix_titles_original.csv


# Suggested code may be subject to a license. Learn more: ~LicenseLog:3096388224.
# https://github.com/drpcc/Labs_solutions.git
# Suggested code may be subject to a license. Learn more: ~LicenseLog:4228235296.
# https://github.com/Aditya2086/Google-Cloud-Skills-Boost.git

# Deploy the first version of the REST API
cd ~/pet-theory/lab06/firebase-rest-api/solution-01  # Navigate to the first solution directory
npm install                                      # Install dependencies for the REST API
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1  # Build and tag the Docker image for the REST API
gcloud beta run deploy $SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 --allow-unauthenticated --region=$REGION  # Deploy the REST API to Cloud Run

# Deploy the second version of the REST API
cd ~/pet-theory/lab06/firebase-rest-api/solution-02  # Navigate to the second solution directory
npm install                                      # Install dependencies for the REST API
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2  # Build and tag the Docker image for the REST API
gcloud beta run deploy $SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 --allow-unauthenticated --region=$REGION  # Deploy the REST API to Cloud Run, overriding the previous version


# Get the URL of the deployed REST API service
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --platform=managed --region=$REGION --format="value(status.url)")

# Test the REST API by making a GET request
curl -X GET $SERVICE_URL/2019

# Update the frontend to use the deployed REST API
cd ~/pet-theory/lab06/firebase-frontend/public  # Navigate to the frontend's public directory

# Comment out the existing REST_API_SERVICE definition in app.js
sed -i 's/^const REST_API_SERVICE = "data\/netflix\.json"/\/\/ const REST_API_SERVICE = "data\/netflix.json"/' app.js

# Add a new REST_API_SERVICE definition pointing to the deployed service
sed -i "1i const
 REST_API_SERVICE = \"$SERVICE_URL/2020\"" app.js


# Build and deploy the frontend
npm install                                     # Install frontend dependencies
cd ~/pet-theory/lab06/firebase-frontend         # Navigate to the frontend directory
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1  # Build and tag the Docker image for the staging frontend
gcloud beta run deploy $FRONTEND_STAGING_SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 --region=$REGION --quiet  # Deploy the staging frontend to Cloud Run

# Build and deploy the production frontend
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1  # Build and tag the Docker image for the production frontend
gcloud beta run deploy $FRONTEND_PRODUCTION_SERVICE_NAME --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1 --region=$REGION --quiet  # Deploy the production frontend to Cloud Run

# source
# Develop Serverless Apps with Firebase: Challenge Lab

# interesting souce in term of presentation:
# Deploy to Kubernetes in Google Cloud: Challenge Lab.md