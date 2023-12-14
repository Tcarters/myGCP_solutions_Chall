
## Diagram Scenario

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/52a5960d-6c76-4255-842f-850de3322c9f)


### Task 1. Ensure that the Pub/Sub API is successfully enabled

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/fef78988-c3b9-4dda-88b0-1dec37434782)


### Task 2. Developing a minimal viable product (MVP)

- Enable the Cloud Run API and configure your Shell environment:

```bash
gcloud services enable run.googleapis.com

## SET LOCATION

LOCATION=us-central1

gcloud config set compute/region $LOCATION

## Deploy the store service

 gcloud run deploy store-service \
  --image gcr.io/qwiklabs-resources/gsp724-store-service \
  --region $LOCATION \
  --allow-unauthenticated

```

### Deploy a consumer service

```bash

## Configure and deploy the order service:

 gcloud run deploy order-service \
  --image gcr.io/qwiklabs-resources/gsp724-order-service \
  --region $LOCATION \
  --no-allow-unauthenticated

```

### Task 3. Deploying Pub/Sub

Now that the producer (store service) and consumer (order service) services have been successfully deployed, you can focus on the main features of Pub/Sub. Using Pub/Sub requires two activities:
- Create a Topic
- Create a Subscription

__Create a Topic__

```bash
gcloud pubsub topics create ORDER_PLACED

```

### Task 4. Creating a service account

To deliver a Pub/Sub message to a Cloud Run service, you need a Pub/Sub subscription. The subscription must be able to invoke the service using a service account with the appropriate permissions. In this lab, the consumer order service will be invoked by a subscription using the service account.

To achieve this functionality, the following activities are required:

- Create a Service Account
- Bind the Invoker Role permissions to the service account

__Service account creation__

```bash
 gcloud iam service-accounts create pubsub-cloud-run-invoker \
  --display-name "Order Initiator"

## COnfirm

 gcloud iam service-accounts list --filter="Order Initiator"
```

__Bind role permissions__

```bash

 gcloud run services add-iam-policy-binding order-service --region $LOCATION \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker --platform managed



## ENV for project
 PROJECT_NUMBER=$(gcloud projects list \
  --filter="qwiklabs-gcp" \
  --format='value(PROJECT_NUMBER)')

## Enable project service account to create tokens
 gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
   --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
   --role=roles/iam.serviceAccountTokenCreator


```


## Task 5. Create a Pub/Sub subscription

```bash
 gcloud pubsub subscriptions create order-service-sub \
   --topic ORDER_PLACED \
   --push-endpoint=$ORDER_SERVICE_URL \
   --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

```

## Task 6. Testing the application

- Create a `test.json` file

```json
{
 "billing_address": {
   "name": "Kylie Scull",
   "address": "6471 Front Street",
   "city": "Mountain View",
   "state_province": "CA",
   "postal_code": "94043",
   "country": "US"
 },
 "shipping_address": {
   "name": "Kylie Scull",
   "address": "9902 Cambridge Grove",
   "city": "Martinville",
   "state_province": "BC",
   "postal_code": "V1A",
   "country": "Canada"
 },
 "items": [
   {
     "id": "RW134",
     "quantity": 1,
     "sub-total": 12.95
   },
   {
     "id": "IB541",
     "quantity": 2,
     "sub-total": 24.5
   }
 ]
}

```

- Create an env variable to store endpoint

```bash
 STORE_SERVICE_URL=$(gcloud run services describe store-service \
   --region $LOCATION \
   --format="value(status.address.url)")
```
- test

```bash
 curl -X POST -H "Content-Type: application/json" -d @test.json $STORE_SERVICE_URL
```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/d97eca1b-2edc-42e2-a96d-2bf01a0969fb)

DOne... 🏴


