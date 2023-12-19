# Continuous Delivery with Google Cloud Deploy


## Task 1. Set variables

```bash
export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-east1
gcloud config set compute/region $REGION
```

## Task 2. Create three GKE clusters

1- Enable API

```bash
gcloud services enable \
container.googleapis.com \
clouddeploy.googleapis.com
```

2 - Create 3 cluters

```

gcloud container clusters create test --node-locations=us-east1-d --num-nodes=1  --async
gcloud container clusters create staging --node-locations=us-east1-d --num-nodes=1  --async
gcloud container clusters create prod --node-locations=us-east1-d --num-nodes=1  --async


### CHECK STATUS

gcloud container clusters list --format="csv(name,status)"

```

## Task 3. Prepare the web application container image

```bash
### ENABLE SERVICE

gcloud services enable artifactregistry.googleapis.com


### CREATE WEB APP REPO
gcloud artifacts repositories create web-app \
--description="Image registry for tutorial web app" \
--repository-format=docker \
--location=$REGION

```

## Task 4. Build and deploy the container images to the Artifact Registry

__Prepare the application configuration__

1. Clone the repository for the lab into your home directory:

```bash

cd ~/
git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
cd cloud-deploy-tutorials
git checkout c3cae80 --quiet
cd tutorials/base
```

2. Create the `skaffold.yaml` configuration:
```
envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
cat web/skaffold.yaml

```
__
The web directory now contains the skaffold.yaml configuration file, which provides instructions for Skaffold to build a container image for your application. This configuration describes the following items.

The build section configures:

The two container images that will be built (artifacts)
The Google Cloud Build project used to build the images
The deploy section configures the Kubernetes manifests needed in deploying the workload to a cluster.

The portForward configuration is used to define the Kubernetes service for the deployment.
__

output

```
cat web/skaffold.yaml
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: skaffold/v2beta7
kind: Config
build:
  artifacts:
    - image: leeroy-web
      context: leeroy-web
    - image: leeroy-app
      context: leeroy-app
  googleCloudBuild:
    projectId: qwiklabs-gcp-01-8067bd02a59c
deploy:
  kubectl:
    manifests:
      - leeroy-web/kubernetes/*
      - leeroy-app/kubernetes/*
portForward:
  - resourceType: deployment
    resourceName: leeroy-web
    port: 8080
    localPort: 9000

```

__Build the web application__

The skaffold tool will handle submission of the codebase to Cloud Build.

- Enable the Cloud Build API:
  
```bash
gcloud services enable cloudbuild.googleapis.com
```

- Run the skaffold command to build the application and deploy the container image to the Artifact Registry repository previously created:

```bash
cd web
skaffold build --interactive=false \
--default-repo $REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--file-output artifacts.json
cd ..

```
- Once the skaffold build has completed, check for the container images in Artifact Registry:

```bash
gcloud artifacts docker images list \
$REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--include-tags \
--format yaml

```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/a6fb0911-dc15-4273-a283-7e97a83a7bfb)

- By default, Skaffold sets the tag for an image to its related git tag if one is available. Similar information can be found in the artifacts.json file that was created by the skaffold command.

Skaffold generates the web/artifacts.json file with details of the deployed images:

```
cat web/artifacts.json | jq
```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/adad9f92-e2ef-40cf-ac16-3b596a8b7164)


## Task 5. Create the delivery pipeline

1. Enable the Google Cloud Deploy API:

```bash

gcloud services enable clouddeploy.googleapis.com

```

2. Create the delivery-pipeline resource using the delivery-pipeline.yaml file:
```bash
gcloud config set deploy/region $REGION
cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
gcloud beta deploy apply --file=clouddeploy-config/delivery-pipeline.yaml

```
3. Verify the delivery pipeline was created:
```bash
gcloud beta deploy delivery-pipelines describe web-app
```

*Note: Notice the first three lines of the output. The delivery pipeline currently references three target environments that haven't been created yet. In the next task you will create those targets.

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/c21933d7-95e4-4ab4-b4e6-e883288d1777)


## Task 6. Configure the deployment targets

Three delivery pipeline targets will be created - one for each of the GKE clusters.

__Ensure that the clusters are ready__

The three GKE clusters should now be running, but it's useful to verify this.

- Run the following to get the status of the clusters:

```bash
gcloud container clusters list --format="csv(name,status)"
```
__Create a context for each cluster__

```
CONTEXTS=("test" "staging" "prod")
for CONTEXT in ${CONTEXTS[@]}
do
    gcloud container clusters get-credentials ${CONTEXT} --region ${REGION}
    kubectl config rename-context gke_${PROJECT_ID}_${REGION}_${CONTEXT} ${CONTEXT}
done
```

__Create a namespace in each cluster__

```
for CONTEXT in ${CONTEXTS[@]}
do
    kubectl --context ${CONTEXT} apply -f kubernetes-config/web-app-namespace.yaml
done
```
![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/8cfff3f2-de7e-40de-9328-91b3d1cec5bb)

The application will be deployed to the (web-app) namespace.



__Create the delivery pipeline targets__

1. Submit a target definition for each of the targets:

```bash
for CONTEXT in ${CONTEXTS[@]}
do
    envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml
    gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml
done

```

2. Display the details for the test Target:

```
cat clouddeploy-config/target-test.yaml


apiVersion: deploy.cloud.google.com/v1beta1
kind: Target
metadata:
  name: test
description: test cluster
gke:
  cluster: projects/qwiklabs-gcp-01-8067bd02a59c/locations/us-east1/clusters/test

```

3. Display the details for the prod Target:

```
cat clouddeploy-config/target-prod.yaml
```

4. Verify the three targets (test, staging, prod) have been created:

```
gcloud beta deploy targets list
```


## Task 7. Create a release

- Since this is the first release of your application, you'll name it web-app-001.

```bash
gcloud beta deploy releases create web-app-001 \
--delivery-pipeline web-app \
--build-artifacts web/artifacts.json \
--source web/
```

- To confirm

```
gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001

### Confirm your application was deployed to the test GKE cluster by running the following commands:

kubectx test
kubectl get all -n web-app
```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/5447bb3d-6a30-4e3d-9a6e-e7dfd6d8a475)


## Task 8. Promote the application to staging

1. Promote the application to the staging target:

```bash
gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001
```

2. To confirm the staging Target has your application deployed, run the following command:

```bash
gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
```


## Task 9. Promote the application to prod

1. Promote the application to the prod target:

```bash
gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001


##### VIEW STATUS

gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001

```

2. Approve the rollout with the following:

```
  gcloud beta deploy rollouts approve web-app-001-to-prod-0001 \
--delivery-pipeline web-app \
--release web-app-001

### Confirm To confirm the prod target has your application deployed, run the following command:


gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001

```

Done... 🎌
