# Serverless Firebase Development: Challenge Lab

- Title: GSP344 Challenge Lab

In this lab you will create a frontend solution using a Rest API and Firestore database. Cloud Firestore is a NoSQL document database that is part of the Firebase platform where you can store, sync, and query data for your mobile and web apps at scale. Lab content is based on resolving a real world scenario through the use of Google Cloud serverless infrastructure.


## Requirements set up


```bash	
	gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')

	gcloud services enable run.googleapis.com

	git clone https://github.com/rosera/pet-theory.git
	
	export REGION=us-central1
	
```

## Task 1. Create a Firestore database

> Cloud Firestore Database
> Use Firestore Native Mode
> Add location Nam5 (United States)


## Task 2. Populate the Database

```bash
	cd pet-theory/lab06/firebase-import-csv/solution
	
	npm install
	
	## Import CSV to Firestore Db
	
	node index.js netflix_titles_original.csv

```

## Task 3. Create a REST API

- Build and Deploy the code to Google Container Registry.

```bash
	cd pet-theory/lab06/firebase-rest-api/solution-01
	
	npm install
	
	gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 
	
```

- Deploy the image as a Cloud Run service.

```bash
	export IMAGE_REST_API="gcr.io/qwiklabs-gcp-01-8a49242b74fe/rest-api:0.1"
	
	export SERVICE_NAME_REST_API="netflix-dataset-service-550"
	
	gcloud beta run deploy $SERVICE_NAME_REST_API --image $IMAGE_REST_API --allow-unauthenticated --region $REGION
	
	export SERVICE_URL=" https://netflix-dataset-service-550-xiqeln7v7q-uc.a.run.app"
	
	curl -X GET $SERVICE_URL
	
```

## Task 4. Firestore API access

- Same process for second version 

```bash
	cd pet-theory/lab06/firebase-rest-api/solution-02
	
	gcloud builds submit --tag=gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 .
	
	export IMAGE_REST_API2="gcr.io/qwiklabs-gcp-01-8a49242b74fe/rest-api:0.2"
	
	export SERV_NAME="netflix-dataset-service-550"
		
	gcloud beta run deploy $SERV_NAME --image $IMAGE_REST_API2 --allow-unauthenticated --region $REGION
	
	export SERVICE_URLv2=" https://netflix-dataset-service-550-xiqeln7v7q-uc.a.run.app"
	
	curl -X GET $SERVICE_URLv2/2019
```

## Task 5 Deploy the STaging Frontend

- Same process as above


## Task 6. Deploy the Production Frontend

```bash
	cd pet-theory/lab06/firebase-frontend/public
	
	sed -i 's/^const REST_API_SERVICE = "data\/netflix\.json"/\/\/ const REST_API_SERVICE = "data\/netflix.json"/' app.js

	sed -i "1i const REST_API_SERVICE = \"$SERVICE_URLv2/2019\" " app.js
	
	## build the image
	gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1
	
	export PROD_IMAGE="gcr.io/qwiklabs-gcp-01-8a49242b74fe/frontend-production:0.1"
	
	
	export FRONTEND_PRODUCTION_SERVICE_NAME=<name>
	
	gcloud beta run deploy $FRONTEND_PRODUCTION_SERVICE_NAME --image $PROD_IMAGE --region=$REGION --quiet
	
```


----------------------------------------------------------------END_CHALLENGE------------------------------------------------------------------------------------




