
## Task1. Create the lab resources

```bash
    export CLUSTER_NAME=hello-cluster
export ZONE=us-central1-b
export REGION=us-central1
export REPO=my-repository
export PROJECT_ID=$(gcloud config get-value project)

```

#### Cretae an Artifact Registry Docker repository named my-repository in the us-central1 region to store your container images.

```bash
    gcloud artifacts repositories create my-repository \
  --repository-format=docker \
  --location=$region
```

### Create a GKE cluster named `hello-cluster`

```bash
     gcloud beta container --project "$PROJECT_ID" clusters create "hello-cluster" --zone "$ZONE" --no-enable-basic-auth --cluster-version latest --release-channel "regular" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true  --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/$PROJECT_ID/global/networks/default" --subnetwork "projects/$PROJECT_ID/regions/$REGION/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --enable-autoscaling --min-nodes "2" --max-nodes "6" --num-nodes "3" --location-policy "BALANCED" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "$ZONE"


      ## Updated with auto scaler
      gcloud container clusters update hello-cluster --enable-autoscaling  --zone us-central1-a

```
- Get Auth of cluster running in GKE
```bash
    	gcloud container clusters get-credentials hello-cluster --zone us-central1-a
```

- Create two namespace

```bash
    kubectl create namespace prod
    kubectl create namespace dev

```


## Task 2. Create a repository in Cloud Source Repositories

1. Create an empty repository named ``sample-app`` in CLoud SOurce Repositories

```bash
        gcloud source repos create sample-app

```

2. Clone the new repo:

```bash
        gcloud source repos clone sample-app

```

3. Copy code to the repo

```bash
    cd ~
    gsutil cp -r gs://spls/gsp330/sample-app/* sample-app

```


## Task3 Create the cloud Build Triggers

```bash
    gcloud builds triggers create cloud-source-repositories --name="sample-app-prod-deploy" --repo="sample-app" --branch-pattern="^master$" --build-config="cloudbuild.yaml"

    ## Second triggers for dev
    gcloud builds triggers create cloud-source-repositories --name="sample-app-dev-deploy" --repo="sample-app" --branch-pattern="^dev$" --build-config="cloudbuild-dev.yaml"


```
## Task 5: Deploy the first versions of the application

- Deploying the devdelopment application

```bash
    sed -i "s/<version>/v1.0/g" cloudbuild-dev.yaml

    sed -i "s/<todo>/${REGION}-docker.pkg.dev/${PROJECT_ID}/$REPO/hello-cloudbuild-dev:v1.0/g"  dev/deployment.yaml

    git add .

    git commit -m "v1.0 on dev branch"

    git push -u origin dev


    ## After suceesfull trigger build
    ## Verify the avail por
    kubectl get deploy -n dev
    
    ## Expose the deploy
    kubectl expose deployment development-deployment --port 8080 --type LoadBalancer -n dev

    kubectl get service -n dev ## to get Ip of deployment 
```


- Deploying the production application

```bash
    
    sed -i "s/<version>/v1.0/g" cloudbuild.yaml
    
    sed -i "s/<todo>/${REGION}-docker.pkg.dev/${PROJECT_ID}/$REPO/hello-cloudbuild:v1.0" prod/deployment.yaml

    git switch master

    git add .

    git commit -m "v1.0 on master"

    git push -u origin master 

    ## After suceesfull trigger build
    ## Verify the avail por
    kubectl get deploy -n prod

      ## Expose the deploy
    kubectl expose deployment production-deployment --port 8080 --type LoadBalancer -n prod


```

## Task 5. Deploy the second versions of the application

### Build the second development deployment


- New content of `` main.go``

```bash
    rm -rf main.go
touch main.go
tee main.go <<EOF
/**
 * Copyright 2023 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package main

import (
	"image"
	"image/color"
	"image/draw"
	"image/png"
	"net/http"
)

func main() {
	http.HandleFunc("/blue", blueHandler)
	http.HandleFunc("/red", redHandler)
	http.ListenAndServe(":8080", nil)
}

func blueHandler(w http.ResponseWriter, r *http.Request) {
	img := image.NewRGBA(image.Rect(0, 0, 100, 100))
	draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{0, 0, 255, 255}}, image.ZP, draw.Src)
	w.Header().Set("Content-Type", "image/png")
	png.Encode(w, img)
}

func redHandler(w http.ResponseWriter, r *http.Request) {
	img := image.NewRGBA(image.Rect(0, 0, 100, 100))
	draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{255, 0, 0, 255}}, image.ZP, draw.Src)
	w.Header().Set("Content-Type", "image/png")
	png.Encode(w, img)
}
EOF

```

- Pushing the code

```bash
    git switch dev

    sed -i "s/v1.0/v2.0/g" cloudbuild-dev.yaml

    sed -i "s/v1.0/v2.0/g" dev/deployment.yaml

    git add .

    git commit -m "v2.0 om dev"

    git push -u origin dev


```

### Same process above for second version of ``production-deployment ``


## Task 6:  Roll back the production deployment

```bash
    kubectl rollout undo deployment/production-deployment  --to-revision=1 -n prod  

```


-----------------------------------------------END OF GAME---------------------------------------------------------------