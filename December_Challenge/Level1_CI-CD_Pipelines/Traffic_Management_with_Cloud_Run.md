# Traffic Management with Cloud Run

## MVP Architecture

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/ebe85341-62cb-4417-b611-1ea4cd01867e)


## Task 1. Configure the environment

```bash
gcloud services enable run.googleapis.com

gcloud services enable run.googleapis.com

LOCATION="us-central1"

```

__Deploy a service__

- Deploy the payment service:


```bash
gcloud run deploy product-service \
   --image gcr.io/qwiklabs-resources/product-status:0.0.1 \
   --tag test1 \
   --region $LOCATION \
   --allow-unauthenticated

```
Output:

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/16124151-cce1-44dc-a612-cd14be7a2a92)


- Add the product status service to an environment variable:

```bash
TEST1_PRODUCT_SERVICE_URL=$(gcloud run services describe product-service --platform managed --region us-central1 --format="value(status.address.url)")

```
- Make a Test: ```curl $TEST1_PRODUCT_SERVICE_URL/help -w "\n"```


## Task 2. Revision tags

- Each new Cloud Run revision can be assigned a tag. Doing this allows access to a URL without serving traffic. An approach like this can be useful to handle the traffic profile across multiple revisions.

- New Architecture

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/ea0a8419-b5d0-450a-abc0-98a9fa468dcd)

### Integration testing

- Deploy a new tagged revision (test2) with redirection of traffic:

```bash
gcloud run deploy product-service \
  --image gcr.io/qwiklabs-resources/product-status:0.0.2 \
  --no-traffic \
  --tag test2 \
  --region=$LOCATION \
  --allow-unauthenticated

```

- Create an environment variable for the new URL:

```bash
TEST2_PRODUCT_STATUS_URL=$(gcloud run services describe product-service --platform managed --region=us-central1 --format="value(status.traffic[1].url)")

## Test :
curl $TEST2_PRODUCT_STATUS_URL/help -w "\n"

```

### Revision migration

- Migrate 50% of the traffic to the revision tag test2:
```bash
gcloud run services update-traffic product-service \
  --to-tags test2=50 \
  --region=$LOCATION
```

- Make confirmnation

```bash
for i in {1..10}; do curl $TEST1_PRODUCT_SERVICE_URL/help -w "\n"; done

```

### Tagged revision rollback

- Migrate the distributed traffic back to the test1 service:

```bash
gcloud run services update-traffic product-service \
  --to-tags test2=0 \
  --region=$LOCATION

## Test

for i in {1..10}; do curl $TEST1_PRODUCT_SERVICE_URL/help -w "\n"; done
```

### Task 3. Traffic migration

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/4b3cb880-d0cf-4e73-bada-b81d0cbfcd6f)

__Traffic Management - Deploy a new version__

- A new Tagged version
```bash

## Version3

gcloud run deploy product-service \
  --image gcr.io/qwiklabs-resources/product-status:0.0.3 \
  --no-traffic \
  --tag test3 \
  --region=$LOCATION \
  --allow-unauthenticated


### Version4
gcloud run deploy product-service \
  --image gcr.io/qwiklabs-resources/product-status:0.0.4 \
  --no-traffic \
  --tag test4 \
  --region=$LOCATION \
  --allow-unauthenticated

```
- Output list of the revision

```bash
LIST=$(gcloud run services describe product-service --platform=managed --region=$LOCATION --format='value[delimiter="=25,"](status.traffic.revisionName)')"=25"
```
- Create an environment variable for the available revisionNames:

```bash
LIST=$(gcloud run services describe product-service --platform=managed --region=$LOCATION --format='value[delimiter="=25,"](status.traffic.revisionName)')"=25"
```

- Split traffic between the four services using the LIST environment variable:

```bash
gcloud run services update-traffic product-service \
  --to-revisions $LIST --region=$LOCATION

```


### Traffic splitting - update traffic between revisions

```bash

gcloud run services update-traffic product-service --to-latest --platform=managed --region=$LOCATION

## Create a new env for new URL:
LATEST_PRODUCT_STATUS_URL=$(gcloud run services describe product-service --platform managed --region=$LOCATION --format="value(status.address.url)")

## Test: curl $LATEST_PRODUCT_STATUS_URL/help -w "\n"

```

Done... 🦋
