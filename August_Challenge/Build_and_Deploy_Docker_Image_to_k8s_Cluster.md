# LAB: Build and Deploy a Docker Image to a Kubernetes Cluster




## Challenge Scenario

> Your development team is interested in adopting a containerized microservices approach to application architecture. You need to test a sample application they have provided for you to make sure that it can be deployed to a Google Kubernetes container. The development group provided a simple Go application called echo-web with a Dockerfile and the associated context that allows you to build a Docker image immediately.

> To test the deployment, you need to download the sample application, then build the Docker container image using a tag that allows it to be stored on the Container Registry. Once the image has been built, you'll push it out to the Container Registry before you can deploy it.

> With the image prepared you can then create a Kubernetes cluster, then deploy the sample application to the cluster.

``Note: In order to ensure accurate lab activity tracking you must use echo-app as the container repository image name, call your Kubernetes cluster echo-cluster, create the Kubernetes cluster in us-east4-c zone and use echo-web for the deployment name.``

## Task 1. Create a Kubernetes Cluster
> Your test environment is limited in capacity, so you should limit the test Kubernetes cluster you are creating to just two e2-standard-2 instances. You must call your cluster echo-cluster.


- Requirement

```bash
    export CLUSTER_NAME=echo-cluster
    export ZONE=us-east4-c
    export REGION=us-east4
    ##export REPO=my-repository
    export PROJECT_ID=$(gcloud config get-value project)
    gcloud config set compute/region $REGION

```

- Create the cluster

```bash
    gcloud container clusters create $CLUSTER_NAME --machine-type e2-standard-2 --num-nodes 2 --zone $ZONE 

```

- Auth to the cluster

```bash
    	gcloud container clusters get-credentials $CLUSTER_NAME

```


## Build a tagged Docker Image

> The sample application, including the Dockerfile and the application context files, are contained in an archive called echo-web.tar.gz. The archive has been copied to a Cloud Storage bucket belonging to your lab project called gs://[PROJECT_ID].

You must deploy this with a tag called v1.

- DOwnload the image from bucket
```bash
    gsutil cp gs://$PROJECT_ID/echo-web.tar.gz .

```

- Build a docker Image

```bash
    cd ~/echo-app
    docker build -t echo-app:v1 .

```

## Task 3. Push the image to the Google Container Registry

> Your organization has decided that it will always use the gcr.io Container Registry hostname for all projects. The sample application is a simple web application that reports some data describing the configuration of the system where the application is running. It is configured to use TCP port 8000 by default.


```bash
    docker tag echo-app:v1 gcr.io/${PROJECT_ID}/echo-app:v1

    docker push gcr.io/qwiklabs-gcp-02-1a82a67d3df1/echo-app:v1

```

## Task 4. Deploy the application to the Kubernetes Cluster

> Even though the application is configured to respond to HTTP requests on port 8000, you must configure the service to respond to normal web requests on port 80. When configuring the cluster for your sample application, call your deployment echo-web


```bash

    ## Create deployment
    kubectl create deployment echo-web --image gcr.io/qwiklabs-gcp-02-1a82a67d3df1/echo-app:v1

    ## Expose the service
    kubectl expose deployment echo-web --target-port 8000 --port 80 --type LoadBalancer 

```
- - -

- Content of Dockerfile

```Dockerfile
    FROM golang:1.8-alpine
    ADD . /go/src/echo-app
    RUN go install echo-app

    FROM alpine:latest
    COPY --from=0 /go/bin/echo-app .
    ENV PORT 8000
    EXPOSE 8080/tcp
    CMD ["./echo-app"]

```

- Content `main.go`
```yaml
/**
 * Copyright 2017 Google Inc.
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

// [START all]
package main

import (
        "fmt"
        "log"
        "net/http"
        "net"
        "strings"
        "os"
)

func main() {
        // use PORT environment variable, or default to 8000
        port := "8000"
        if fromEnv := os.Getenv("PORT"); fromEnv != "" {
                port = fromEnv
        }

        // register hello function to handle all requests
        server := http.NewServeMux()
        server.HandleFunc("/", echo)

        // start the web server on port and accept requests
        log.Printf("Server listening on port %s", port)
        err := http.ListenAndServe(":"+port, server)
        log.Fatal(err)
}

// echo responds to the request with a plain-text "Serving request" message 
// followed by some meta-data baout the environment where it is running
func echo(w http.ResponseWriter, r *http.Request) {
        log.Printf("Serving request: %s", r.URL.Path)
        host, _ := os.Hostname()
        addrs, err := net.LookupHost(host)
        ipaddresses := ""

        if err == nil {
                ipaddresses = strings.Join(addrs, " ")
        }

        fmt.Fprintf(w, "Echo Test\n")
        fmt.Fprintf(w, "Version: 1.0.0\n")
        fmt.Fprintf(w, "Hostname: %s\n", host)
        fmt.Fprintf(w, "Host ip-address(es): %s\n", ipaddresses)
}
// [END all]

```


---------------------------------------------_END CHALLENGE----------------------------------------------