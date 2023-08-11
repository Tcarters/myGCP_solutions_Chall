# Deploy Your Website on Cloud Run

With Cloud Run, Google Cloud's implementation of Google's Knative framework, you can manage and deploy your website without any of the infrastructure overhead you experience with a VM or pure Kubernetes-based deployments. Not only is this a simpler approach from a management perspective, it also gives you the ability to "scale to zero" when there are no requests coming into your website.

Cloud Run brings "serverless" development to containers and can be run either on your own Google Kubernetes Engine (GKE) clusters or on a fully managed PaaS solution provided by Cloud Run. You will be running the latter scenario in this lab.

The exercises are ordered to reflect a common cloud developer experience:

1. Create a Docker container from your application

2. Deploy the container to Cloud Run

3. Modify the website

4. Roll out a new version with zero downtime


## Task 1. Clone the source repository

```bash
    git clone https://github.com/googlecodelabs/monolith-to-microservices.git
    cd ~/monolith-to-microservices

    ./setup.sh

    cd ~/monolith-to-microservices/monolith
    npm start


```

## Task 2. Create a Docker container with Cloud Build

Now that you have the source files ready to go, it is time to Dockerize your application!

Normally you would have to take a two step approach that entails building a docker container and pushing it to a registry to store the image for GKE to pull from. Make life easier and use Cloud Build to build the Docker container and put the image in Artifact Registry with a single command!


### Create the target Docker repository

### Configure authentication

```bash
    gcloud auth configure-docker us-central1-docker.pkg.dev

```

### Deploy the image

- Enable API 

```bash
    gcloud services enable artifactregistry.googleapis.com \
    cloudbuild.googleapis.com \
    run.googleapis.com

```

- Start Builds
```bash
    gcloud builds submit --tag us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0
```

### Task 3. Deploy the container to Cloud Run

- Run the following command to deploy the image to Cloud Run:

```bash
    gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1

    ## Verify deployment
    gcloud run services list

```

## Task 4. Create new revision with lower concurrency

- Run the following command to re-deploy the same container image with a concurrency value of 1 (just for testing), and see what happens:
```bash
    gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1 --concurrency 1


    ## Run the following command to update the current revision, using a concurrency value of 80:

    gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-central1 --concurrency 80
```

## Task 5. Make changes to the website

```bash
    cd ~/monolith-to-microservices/react-app/src/pages/Home
    mv index.js.new index.js

    ## Rebuild the app 
    cd ~/monolith-to-microservices/react-app
    npm run build:monolith

    ## Trigger a new Cloud Build with a new image 2.0.0
    cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:2.0.0

```

## Task 6. Update website with zero downtime

```bash
    gcloud run deploy monolith --image us-central1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:2.0.0 --region us-central1

    ## verify Deployment
    gcloud run services describe monolith --platform managed --region us-central1

    ## List services
    gcloud beta run services list

```


---------------------------------------------------_END_GAME----------------------------