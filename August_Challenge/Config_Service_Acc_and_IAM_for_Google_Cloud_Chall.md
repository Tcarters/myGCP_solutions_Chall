

Task 1. Create a service account using the gcloud CLI

- After login on SSH in VM , set up :

```bash

    gcloud auth login

    gcloud config set project qwiklabs-gcp-02-33a047d26236

    gcloud config set compute/region us-west1
    
    gcloud config set compute/zone us-west1-a
```

- Create service Account `devops`

```bash
     
     gcloud iam service-accounts create devops --display-name "my devops service account"

```

## Task 2. Grant IAM permissions to a service account using the gcloud CLI


```bash
    export PROJECT_ID=$(gcloud config get-value project)

    export SA=$(gcloud iam service-accounts list --format="value(email)" --filter "displayName=devops")

    gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$SA --role=roles/iam.serviceAccountUser

    gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$SA --role=roles/compute.instanceAdmin

```

## Task 3. Create a compute instance with a service account attached using gcloud

- Create a compute instance named vm-2 with the devops service account attached that you created in Task 2.

- SSH into the vm-2 VM instance. Try to create and list an instance from vm-2 to verify you have the necessary permissions via the service account.



```bash
    export ZONE=us-west1-a

    gcloud compute instances create vm-2 --service-account=$SA  --zone=$ZONE

```

## Task 4. Create a custom role using a YAML file

```bash
    cat > role-definition.yaml <<EOF
title: Custom Role
description: Custom role with cloudsql.instances.connect and cloudsql.instances.get permissions
includedPermissions:
- cloudsql.instances.connect
- cloudsql.instances.get
EOF

    gcloud iam roles create customRole --project=$PROJECT_ID --file=role-definition.yaml

```

## Task 5. Use the client libraries to access BigQuery from a service account

- Create service Account `bigquery-qwiklab`

```bash

    gcloud iam service-accounts create bigquery-qwiklab --display-name bigquery-qwiklab

    export SA2=$(gcloud iam service-accounts list --format="value(email)" --filter "displayName=bigquery-qwiklab")

    gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$SA2 --role=roles/bigquery.dataViewer

    gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$SA2 --role=roles/bigquery.user


```

- Create an Instance with Service Account

```bash
    gcloud compute instances create bigquery-instance --service-account=$SA2 --scopes=https://www.googleapis.com/auth/bigquery --zone=$ZONE

```


### SSH into the bigquery-instance and install dependencies; use the following code to create a Python file:

- content of query.app

```python3
    echo "
from google.auth import compute_engine
from google.cloud import bigquery
credentials = compute_engine.Credentials(
    service_account_email='YOUR_SERVICE_ACCOUNT')
query = '''
SELECT name, SUM(number) as total_people
FROM "bigquery-public-data.usa_names.usa_1910_2013"
WHERE state = 'TX'
GROUP BY name, state
ORDER BY total_people DESC
LIMIT 20
'''
client = bigquery.Client(
    project='YOUR_PROJECT_ID',
    credentials=credentials)
print(client.query(query).to_dataframe())
" > query.py

```

- Install dependencies

```bash
    sudo apt-get install -y git python3-pip
    pip3 install --upgrade pip
    pip3 install google-cloud-bigquery
    pip3 install pyarrow pandas db-types
```

- Update app with SA and PROJECT_ID
```bash

    sed -i -e "s/Your Project ID/$(gcloud config get-value project)/g" query.py

    ###Add the service account email to query.py with:

    sed -i -e "s/YOUR_SERVICE_ACCOUNT/bigquery-qwiklab@$(gcloud config get-value project).iam.gserviceaccount.com/g" query.py


    ## Run app
    python3 query.py

```

-----------------------------------------------------------END_CHALLENGE------------------------------------------------------------