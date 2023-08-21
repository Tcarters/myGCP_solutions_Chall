# Cloud Scheduler: Qwik Start


## Task 1. Enable Cloud Scheduler API


## Task 2. Set up Cloud Pub/Sub

```bash
	gcloud config set compute/region us-east1
	
	gcloud pubsub topics create cron-topic
	## This command creates a topic called cron-topic
	
	
	## Create a Cloud Pub/Sub subscription:
	gcloud pubsub subscriptions create cron-sub --topic cron-topic
	
```

## Task 3. Create a job

```bash
	
```

## Task 4. Verify the results in Cloud Pub/Sub

```bash
	##To verify that your Cloud Pub/Sub topic is receiving messages from your job, invoke the following command:
	 gcloud pubsub subscriptions pull cron-sub --limit 5
	
```

----------------------------------------------------------------------------END-GAME_-----------------------------------------------------------------------------

