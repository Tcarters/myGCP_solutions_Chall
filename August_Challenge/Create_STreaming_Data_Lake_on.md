# Create a Streaming Data Lake on Cloud Storage: Challenge Lab


```bash

export BUCKET_NAME=qwiklabs-gcp-04-70f36c8608bb-bucket

export TOPIC_ID=mypubsub

export PROJECT_ID=qwiklabs-gcp-04-70f36c8608bb

export REGION="us-central1"

export AE_REGION=us-central

gcloud config set compute/region $REGION


gcloud services enable dataflow.googleapis.com cloudscheduler.googleapis.com


PROJECT_ID=$(gcloud config get-value project)

```


## Task 1. Create a Pub/Sub topic


```bash
	gcloud pubsub topics create $TOPIC_ID
```


## Task 2. Create a Cloud Scheduler job

```bash
	### Create an App Engine
	
	gcloud app create --region=$AE_REGION
	
	## Create a Cloud scheduler 
	
	gcloud scheduler jobs create pubsub publisher-job --schedule="* * * * *" \
    --topic=$TOPIC_ID --message-body="Hello!"
    
    	## Start the Job
    	
    	gcloud scheduler jobs run publisher-job
```

## Task 3. Create a Cloud Storage bucket

```bash
	gsutil mb gs://$BUCKET_NAME
```


## Task 4. Run a Dataflow pipeline to stream data from a Pub/Sub topic to Cloud Storage


```bash
	git clone https://github.com/GoogleCloudPlatform/java-docs-samples.git
	cd java-docs-samples/pubsub/streaming-analytics

	## Start the pipeline

		mvn compile exec:java \
		-Dexec.mainClass=com.examples.pubsub.streaming.PubSubToGcs \
		-Dexec.cleanupDaemonThreads=false \
		-Dexec.args=" \
		--project=$PROJECT_ID \
		--region=$REGION \
		--inputTopic=projects/$PROJECT_ID/topics/$TOPIC_ID \
		--output=gs://$BUCKET_NAME/samples/output \
		--runner=DataflowRunner \
		--windowSize=2"
    
```




----------------------------------------------------------🎉END-GAME🎉-------------------------------------------------------------------------------------------
