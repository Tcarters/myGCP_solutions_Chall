# Google Kubernetes Engine Pipeline using Cloud Build

In this lab, you create a CI/CD pipeline that automatically builds a container image from committed code, stores the image in Artifact Registry, updates a Kubernetes manifest in a Git repository, and deploys the application to Google Kubernetes Engine using that manifest.

## Task 1. Initialize Your Lab

- In Cloud Shell, set your project ID and project number. Save them as PROJECT_ID and PROJECT_NUMBER variables:

```bash
    export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION=us-central1
gcloud config set compute/region $REGION

```

-2. Enable APIs for for GKE, Cloud Build, Cloud Source Repositories and Container Analysis:

```bash
    gcloud services enable container.googleapis.com \
    cloudbuild.googleapis.com \
    sourcerepo.googleapis.com \
    containeranalysis.googleapis.com
```

- 3. Create an Artifact Registry Docker repository named my-repository in the us-central1 region to store your container images:

```bash
    gcloud artifacts repositories create my-repository \
  --repository-format=docker \
  --location=$REGION

```

- 4. Create a GKE cluster to deploy the sample application

```bash
      gcloud container clusters create hello-cloudbuild --num-nodes 1 --region $REGION
```

- 5. Configure Git credentials

```bash
    git config --global user.email "you@example.com"  
    git config --global user.name "Your Name"
```

## Task 2. Create the Git repositories in Cloud Source Repositories

- Create two Git repositories to GCP repositories

```bash
    gcloud source repos create hello-cloudbuild-app

    gcloud source repos create hello-cloudbuild-env

```
- Clone the sample code from GitHub:

```bash
    cd ~

    git clone https://github.com/GoogleCloudPlatform/gke-gitops-tutorial-cloudbuild hello-cloudbuild-app

```
- Configure Cloud Source Repositories as a remote:

```bash

    cd ~/hello-cloudbuild-app

    PROJECT_ID=$(gcloud config get-value project)

    git remote add google "https://source.developers.google.com/p/${PROJECT_ID}/r/hello-cloudbuild-app"

```


## Task 3. Create a container image with Cloud Build
With this Dockerfile, you can create a container image with Cloud Build and store it in Artifact Registry.

- In Cloud Shell, create a Cloud Build build based on the latest commit with the following command:

```bash
    cd ~/hello-cloudbuild-app

    COMMIT_ID="$(git rev-parse --short=7 HEAD)"

    gcloud builds submit --tag="${REGION}-docker.pkg.dev/${PROJECT_ID}/my-repository/hello-cloudbuild:${COMMIT_ID}" .

```


## Task 4. Create the Continuous Integration (CI) pipeline

```Text
    In the Cloud console, go to Cloud Build > Triggers.
1. Click Create Trigger
2. In the Name field, type hello-cloudbuild.
3. Under Event, select Push to a branch.
4. Under Source, select hello-cloudbuild-app as your Repository and .* (any branch) as your Branch.
5. Under Build configuration, select Cloud Build configuration file.
6. In the Cloud Build configuration file location field, type cloudbuild.yaml after the /.
7. Click Create.

```

## Task5 . Create the Test Environment and CD pipeline

### Grant Cloud Build access to GKE

To deploy the application in your Kubernetes cluster, Cloud Build needs the Kubernetes Engine Developer Identity and Access Management role.


```bash
    PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"

    gcloud projects add-iam-policy-binding ${PROJECT_NUMBER} \
--member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
--role=roles/container.developer

```

- You need to initialize the hello-cloudbuild-env repository with two branches (production and candidate) and a Cloud Build configuration file describing the deployment process.


- Step 1: clone repo

```bash
    cd ~

    gcloud source repos clone hello-cloudbuild-env

    cd ~/hello-cloudbuild-env
```

- Next you need to copy the cloudbuild-delivery.yaml file available in the hello-cloudbuild-app repository and commit the change:

```bash
    cd ~/hello-cloudbuild-env

    cp ~/hello-cloudbuild-app/cloudbuild-delivery.yaml ~/hello-cloudbuild-env/cloudbuild.yaml

    git add .

    git commit -m "Create cloudbuild.yaml for deployment"


```
- The ``cloudbuild-delivery.yaml`` file describes the deployment process to be run in Cloud Build. It has two steps:
. Cloud Build applies the manifest on the GKE cluster.
. If successful, Cloud Build copies the manifest on the production branch.



- Create a candidate branch

```bash
    
    git checkout -b candidate

    git push origin production

    git push origin candidate

```

- 5. Grant the SOurce Repository Writer IAM role to CLoud Build

```bash
    PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} \
--format='get(projectNumber)')"
cat >/tmp/hello-cloudbuild-env-policy.yaml <<EOF
bindings:
- members:
  - serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
  role: roles/source.writer
EOF



gcloud source repos set-iam-policy \
hello-cloudbuild-env /tmp/hello-cloudbuild-env-policy.yaml

```

- Create the Cloud Build trigger ``hello-cloudbuild-deploy`` for the continuous delivery pipeline
