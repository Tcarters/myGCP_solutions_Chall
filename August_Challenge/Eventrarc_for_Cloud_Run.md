# Eventarc for Cloud Run


## Task 1. Set up your environment

```bash
    gcloud config set project qwiklabs-gcp-00-5bf8ef91ac68

    gcloud config set run/region us-east1

    ## Set the Cloud Run platform default to managed:

    gcloud config set run/platform managed

    ## Set the location default of Eventarc for Cloud Run:
    gcloud config set eventarc/location us-east1


```

## Task 2. Enable service account


```bash
    ## Store the Project Number in an environment variable:


    export PROJECT_NUMBER="$(gcloud projects list \
  --filter=$(gcloud config get-value project) \
  --format='value(PROJECT_NUMBER)')"

    ## Grant the eventarc.admin role to the default Compute Engine service account
    gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
  --member=serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com \
  --role='roles/eventarc.admin'


```


## Task 3. Event discovery

```bash
        ## List events
         gcloud beta eventarc attributes types list


        ## Get more info avbout an event
        gcloud beta eventarc attributes types describe \
  google.cloud.pubsub.topic.v1.messagePublished
```


## Task 4. Create a Cloud Run sink

```bash

    export SERVICE_NAME=event-display

    ## Set up an environment variable for the image:

    export IMAGE_NAME="gcr.io/cloudrun/hello"

    ## Deploy your containerized application to Cloud Run:

    gcloud run deploy ${SERVICE_NAME} \
  --image ${IMAGE_NAME} \
  --allow-unauthenticated \
  --max-instances=3



```

## Task 5. Create a Cloud Pub/Sub event trigger

- Create a trigger

```bash

    ## Get more info about a trigger
    gcloud beta eventarc attributes types describe \
  google.cloud.pubsub.topic.v1.messagePublished

    ## Create a trigger to filter events published to the Cloud Pub/Sub topic to your deployed Cloud Run service:


    gcloud beta eventarc triggers create trigger-pubsub \
  --destination-run-service=${SERVICE_NAME} \
  --matching-criteria="type=google.cloud.pubsub.topic.v1.messagePublished"

```

- Find the topic

```bash
    export TOPIC_ID=$(gcloud eventarc triggers describe trigger-pubsub \
  --format='value(transport.pubsub.topic)')

```

- Test the trigger

```bash
    gcloud eventarc triggers list

    ## to simulate a custom application sending message, you can use a gcloud command to to fire an event
    gcloud pubsub topics publish ${TOPIC_ID} --message="Hello there"

```

- Delete the trigger

```bash
    gcloud eventarc triggers delete trigger-pubsub
```


## Task 6. Create a Audit Logs event trigger

- Create a bucket


```bash
    export BUCKET_NAME=$(gcloud config get-value project)-cr-bucket

    gsutil mb -p $(gcloud config get-value project) \
  -l $(gcloud config get-value run/region) \
  gs://${BUCKET_NAME}/


```

> Enable Audit Logs
1. In order to receive events from a service, you need to enable audit logs.

2. From the Navigation menu, select IAM & Admin > Audit Logs.

3. In the list of services, check the box for Google Cloud Storage.

4. On the right hand side, click the LOG TYPE tab. Admin Write is selected by default, make sure you also select Admin Read, Data Read, Data Write and then click Save.


- Test audit logs

```bash
    echo "Hello World" > random.txt

    gsutil cp random.txt gs://${BUCKET_NAME}/random.txt

```

- Create another event

```bash
    gcloud beta eventarc attributes types describe google.cloud.audit.log.v1.written

    gcloud beta eventarc triggers create trigger-auditlog \
--destination-run-service=${SERVICE_NAME} \
--matching-criteria="type=google.cloud.audit.log.v1.written" \
--matching-criteria="serviceName=storage.googleapis.com" \
--matching-criteria="methodName=storage.objects.create" \
--service-account=${PROJECT_NUMBER}-compute@developer.gserviceaccount.com

```