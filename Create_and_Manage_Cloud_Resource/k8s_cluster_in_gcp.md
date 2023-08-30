## Create a GKE Cluster

```bash
	gcloud container clusters create --machine-type=e2-medium --zone=us-east1-d lab-cluster --num-nodes 1
```

## Get Authentication credentials for the cluster

- Authenticate with the cluster

```bash
	gcloud container clusters get-credentials lab-cluster 
	
```

## Deploy an application to cluster

- Create a new deploymnet
```bash
	kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
	
```
- Create a k8s service

```bash
	kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

## Delete a cluster

```bash
	gcloud container clusters delete lab-cluster 
	
	kubectl get service
	
```
