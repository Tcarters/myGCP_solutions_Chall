
## Enable the Dataplex API and set variables

```bash
	gcloud services enable \
  dataplex.googleapis.com 
  
  	export PROJECT_ID=$(gcloud config get-value project)
  	
  	export REGION=us-central1
	gcloud config set compute/region $REGION
	
```

## Task 1. Create a lake

```bash
	gcloud dataplex lakes create ecommerce \
   --location=$REGION \
   --display-name="Ecommerce" \
   --description="Ecommerce Domain"
   	
```

## Task 2. Add a zone to your lake

```bash
	gcloud dataplex zones create orders-curated-zone \
    --location=$REGION \
    --lake=ecommerce \
    --display-name="Orders Curated Zone" \
    --resource-location-type=SINGLE_REGION \
    --type=CURATED \
    --discovery-enabled \
    --discovery-schedule="0 * * * *"
    
```

## Task 3. Attach an asset to a zone

- Create a BigQuery dataset :

```bash
	bq mk --location=$REGION --dataset orders 
```

- Attach the BigQuery dataset to the zone

```bash
	gcloud dataplex assets create orders-curated-dataset \
--location=$REGION \
--lake=ecommerce \
--zone=orders-curated-zone \
--display-name="Orders Curated Dataset" \
--resource-type=BIGQUERY_DATASET \
--resource-name=projects/$PROJECT_ID/datasets/orders \
--discovery-enabled 

```

## Task 4. Delete assets, zones, and lakes

- Detach an asset

```bash
	gcloud dataplex assets delete orders-curated-dataset --location=$REGION --zone=orders-curated-zone --lake=ecommerce 
```

- Delete a zone

```bash
		gcloud dataplex zones delete orders-curated-zone --location=$REGION --lake=ecommerce
```

- Delete the lake

```bash
	gcloud dataplex lakes delete ecommerce --location=$REGION
	
```

------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------------------
