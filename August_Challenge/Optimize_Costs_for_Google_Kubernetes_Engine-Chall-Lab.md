
# Optimize Costs for Google Kubernetes Engine: Challenge Lab



## Task 1. Create our cluster and deploy our app


```bash
	gcloud config set compute/region us-central1
		
	gcloud container clusters create "onlineboutique-cluster-126" --release-channel "regular" --machine-type=e2-standard-2 --num-nodes 2 --zone us-central1-a
	
	gcloud container clusters get-credentials "onlineboutique-cluster-126" --zone us-central1-b --project ${GOOGLE_CLOUD_PROJECT}	
	
	kubectl create namespace dev

	kubectl create namespace prod
	
	git clone https://github.com/GoogleCloudPlatform/microservices-demo.git &&
cd microservices-demo && kubectl apply -f ./release/kubernetes-manifests.yaml --namespace dev

```

## Task 2. Migrate to an optimized node pool

```bash
	export CLUSTER_NAME="onlineboutique-cluster-713"
	export POOLNAME="optimized-pool-9017"
	export ZONE="us-central1-b"
	gcloud container node-pools create $POOLNAME --cluster=$CLUSTER_NAME --machine-type="custom-2-3584" --num-nodes=2 --zone=us-central1-b
	
	for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do  kubectl cordon "$node"; done

	for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node"; done

	## Verifying running pod
	
	kubectl get pods -o=wide --namespace=dev

	## Delete Node-pools
	
	gcloud container node-pools delete default-pool --cluster $CLUSTER_NAME --zone $ZONE --quiet



```


## Task 3. Apply a frontend update


```bash
	kubectl create poddisruptionbudget onlineboutique-frontend-pdb --selector app=frontend --min-available 1 --namespace dev

	## Updating frontend
	
	kubectl patch deployment frontend -n dev --type=json -p '[
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/image",
    "value": "gcr.io/qwiklabs-resources/onlineboutique-frontend:v2.1"
  },
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/imagePullPolicy",
    "value": "Always"
  }
]'

```


## Task 4. Autoscale from estimated traffic

```bash
	export MAX_REPLICAS=12
	kubectl autoscale deployment frontend --cpu-percent=50 --min=1 --max=$MAX_REPLICAS --namespace dev
	
	## Verifying Autoscaling:
	
	kubectl get hpa --namespace dev

	## Autoscale Cluster nodes to 6
	
	gcloud beta container clusters update $CLUSTER_NAME --enable-autoscaling --min-nodes 1 --max-nodes 6 --zone=$ZONE


```


## OPTIONAL

```bash
	kubectl exec $(kubectl get pod --namespace=dev | grep 'loadgenerator' | cut -f1 -d ' ') -it --namespace=dev -- bash -c 'export USERS=8000; locust --host="http://34.135.152.136" --headless -u "8000" 2>&1'
```

------------------------------------------------------🎉_CHALLENGE_OVER_🎉-------------------------------------------------------------------------------------


