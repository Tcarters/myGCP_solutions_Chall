# Cloud Run Canary Deployments


## Objectives
Create your Cloud Run service.
Enable developer branch.
Implement canary testing.
Rollout safely to production.

## Task 1. Preparing your environment

- Create env
  
```bash
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION=us-east4
gcloud config set compute/region $REGION
```

- Enable the following APIs with the code below:
Cloud Resource Manager
GKE
Cloud Source Repositories
Cloud Build
Container Registry
Cloud Run

```bash
gcloud services enable \
cloudresourcemanager.googleapis.com \
container.googleapis.com \
sourcerepo.googleapis.com \
cloudbuild.googleapis.com \
containerregistry.googleapis.com \
run.googleapis.com

```

- Grant the Cloud Run Admin role (roles/run.admin) to the Cloud Build service account:

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
--role=roles/run.admin

```
- Grant the IAM Service Account User role (roles/iam.serviceAccountUser) to the Cloud Build service account for the Cloud Run runtime service account:

```bash
gcloud iam service-accounts add-iam-policy-binding \
$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
--member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
--role=roles/iam.serviceAccountUser
```

- COde

```bash
git clone https://github.com/GoogleCloudPlatform/software-delivery-workshop --branch cloudrun-progression-csr cloudrun-progression
cd cloudrun-progression/labs/cloudrun-progression
rm -rf ../../.git

```
- Push

```bash
gcloud source repos create cloudrun-progression
git init
git config credential.helper gcloud.sh
git remote add gcp https://source.developers.google.com/p/$PROJECT_ID/r/cloudrun-progression
git branch -m master
git add . && git commit -m "initial commit"
git push gcp master
```

## Task 2. Creating your Cloud Run service

```bash
gcloud builds submit --tag gcr.io/$PROJECT_ID/hello-cloudrun
gcloud run deploy hello-cloudrun \
--image gcr.io/$PROJECT_ID/hello-cloudrun \
--platform managed \
--region $REGION \
--tag=prod -q
```

- View the authenticated service:

```bash
PROD_URL=$(gcloud run services describe hello-cloudrun --platform managed --region $REGION --format=json | jq --raw-output ".status.url")
echo $PROD_URL
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $PROD_URL
```

## Task 3. Enabling Dynamic Developer Deployments

- Set a trigger

```bash
gcloud beta builds triggers create cloud-source-repositories --trigger-config branch-trigger.json

## Swich branch
git checkout -b new-feature-1
```
- Modify code:

You can now browse or modify the code with the editor. In the sample application (~/cloudrun-progression/labs/cloudrun-progression/app.py), modify line 24 to indicate v1.1 instead of v1.0:

```bash
@app.route('/')
def hello_world():
return 'Hello World v1.1'
```
- Push the code
```bash
git add . && git commit -m "updated" && git push gcp new-feature-1
```

- Get uniq URL

``BRANCH_URL=$(gcloud run services describe hello-cloudrun --platform managed --region $REGION --format=json | jq --raw-output ".status.traffic[] | select (.tag==\"new-feature-1\")|.url")
echo $BRANCH_URL``

- Access it :

``curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $BRANCH_URL``

## Task 4. Automating canary testing

1.In Cloud Shell, set up the branch trigger:

```bash
gcloud beta builds triggers create cloud-source-repositories --trigger-config master-trigger.json
```

2. Get the uniq URL

```bash
CANARY_URL=$(gcloud run services describe hello-cloudrun --platform managed --region $REGION --format=json | jq --raw-output ".status.traffic[] | select (.tag==\"canary\")|.url")
echo $CANARY_URL

curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $CANARY_URL
```
- See

```bash
LIVE_URL=$(gcloud run services describe hello-cloudrun --platform managed --region $REGION --format=json | jq --raw-output ".status.url")
for i in {0..20};do
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $LIVE_URL; echo \n
done
```

## Task 5. Releasing to Production

- After the canary deployment is validated with a small subset of traffic, you release the deployment to the remainder of the live traffic.

In this section, you set up a trigger that is activated when you create a tag in the repository. The trigger migrates 100% of traffic to the already deployed revision based on the commit SHA of the tag. Using the commit SHA ensures the revision validated with canary traffic is the revision utilized for the remainder of production traffic.

```bash
gcloud beta builds triggers create cloud-source-repositories --trigger-config tag-trigger.json
```
- Create a new Tag

```bash
git tag 1.1
git push gcp 1.1

```

- In Cloud Shell, to see percentage-based responses, make a series of requests:

```bash
LIVE_URL=$(gcloud run services describe hello-cloudrun --platform managed --region $REGION --format=json | jq --raw-output ".status.url")
for i in {0..20};do
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $LIVE_URL; echo \n
done

```

Done...
