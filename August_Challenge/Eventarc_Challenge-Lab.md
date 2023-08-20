# Get Started with Eventarc: Challenge Lab


## Task 1. Create a Pub/Sub topic


- Enable services

```bash
	export REGION=us-central1	
	
	gcloud services enable \
  artifactregistry.googleapis.com \
  cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com \
  eventarc.googleapis.com \
  run.googleapis.com \
  logging.googleapis.com \
  pubsub.googleapis.com
  
```

- Create a Pub/Sub topic named qwiklabs-gcp-02-33ae7bf1d2d0-topic with a subscription named qwiklabs-gcp-02-33ae7bf1d2d0-topic-sub.

```bash
	## Create a Topic
	gcloud pubsub topics create "qwiklabs-gcp-02-33ae7bf1d2d0-topic"
	
	## Create a subscription to topic
	
	gcloud pubsub subscriptions create --topic qwiklabs-gcp-02-33ae7bf1d2d0-topic qwiklabs-gcp-02-33ae7bf1d2d0-topic-sub
```



## Task 2. Create a Cloud Run sink

```bash

	export SERVICE_NAME=pubsub-events

	export IMAGE_NAME="gcr.io/cloudrun/hello"
	
	
	## Deploy our containerized app
	gcloud run deploy ${SERVICE_NAME} \
  --image ${IMAGE_NAME} \
  --allow-unauthenticated \
  --max-instances=3

```


## Task 3. Create and test a Pub/Sub event trigger using Eventarc

```bash
	## Create an event Trigger
	
	gcloud eventarc triggers create pubsub-events-trigger \
  --location=$REGION \
  --destination-run-service=pubsub-events \
  --destination-run-region=$REGION \
  --transport-topic=$DEVSHELL_PROJECT_ID-topic \
  --event-filters="type=google.cloud.pubsub.topic.v1.messagePublished"
  
  	## Publish a message:
  	gcloud pubsub topics publish $DEVSHELL_PROJECT_ID-topic --message="Hello there"

  	
  	## Checking:
  	
  	gcloud eventarc triggers list
```


---------------------------------------------------------------------------🎉GAME_OVER_🎉------------------------------------------------------------------------
