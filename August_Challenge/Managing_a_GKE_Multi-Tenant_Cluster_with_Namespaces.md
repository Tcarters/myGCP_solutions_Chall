# Managing a GKE Multi-tenant Cluster with Namespaces


## Task 1. Download required files


```bash
	gsutil -m cp -r gs://spls/gsp766/gke-qwiklab ~
	
	cd ~/gke-qwiklab
	
```

## Task 2. View and create namespaces

```bash
	export ZONE=us-central1-f
	gcloud config set compute/zone ${ZONE} && gcloud container clusters get-credentials multi-tenant-cluster


	## Default namespaces
	
	kubectl get namespace
	
	## List of namespaced
	kubectl api-resources --namespaced=true
	
	
	kubectl get services --namespace=kube-system
```

- Creating new namespaces

```bash
	kubectl create namespace team-a && \
	kubectl create namespace team-b
	
	# Start a pods
	
	kubectl run app-server --image=centos --namespace=team-a -- sleep infinity && \ kubectl run app-server --image=centos --namespace=team-b -- sleep infinity
	
	## Get all pods
	
	kubectl get pods -A
	
	
	## See additional details 
	kubectl describe pod app-server --namespace=team-a
	
	##Set a context to work exclusively with one namespace
	
	kubectl config set-context --current --namespace=team-a

	kubectl describe pod app-server
```


## Task 3. Access Control in namespaces

- Grant the account the Kubernetes Engine Cluster Viewer role by running the following:


```bash
	gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
--member=serviceAccount:team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com  \
--role=roles/container.clusterViewer
```

### K8s RBAC

1. Roles with single rules can be created with kubectl create:

```bash
	kubectl create role pod-reader \
--resource=pods --verb=watch --verb=get --verb=list
```

2. Create roles using file

```yaml
	cat developer-role.yaml
	
	## Apply roles
	kubectl create -f developer-role.yaml
	
	## Create a role binding between the team-a-developers serviceaccount and the developer-role:
	
	kubectl create rolebinding team-a-developers \
--role=developer --user=team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
```

3. Test the rolebinding

```bash
	## Download the service account keys used to impersonate the service account:
	
	gcloud iam service-accounts keys create /tmp/key.json --iam-account team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
	
	## Activate the service account
	
	gcloud auth activate-service-account  --key-file=/tmp/key.json
	
	## Get credentials for cluster
	
	export ZONE=us-central1-f
	gcloud container clusters get-credentials multi-tenant-cluster --zone ${ZONE} --project ${GOOGLE_CLOUD_PROJECT}
	
	kubectl get pods --namespace=team-a
```


## Task 4. Resource quotas

```bash
	kubectl create quota test-quota \
--hard=count/pods=2,count/services.loadbalancers=1 --namespace=team-a

	kubectl run app-server-2 --image=centos --namespace=team-a -- sleep infinity
	
	# Create a 3rd pods will fails
	kubectl run app-server-3 --image=centos --namespace=team-a -- sleep infinity
	
	## Details about resource quota
	
	kubectl describe quota test-quota --namespace=team-a
	
	## Update `test-quota` to limit of 6 pods
	
	export KUBE_EDITOR="nano"
kubectl edit quota test-quota --namespace=team-a

```

- CPU and memory quotas

```bash
	kubectl create -f cpu-mem-quota.yaml
	
	## Create a pod with cpu & memory 
	
	kubectl create -f cpu-mem-demo-pod.yaml --namespace=team-a
	
	kubectl describe quota cpu-mem-quota --namespace=team-a
	
```

## Task 5. Monitoring GKE and GKE usage metering

```bash

	## Enable GKE usage metering on the cluster:
	
	export ZONE=us-central1-f
	gcloud container clusters \
  		update multi-tenant-cluster --zone ${ZONE} \
  		--resource-usage-bigquery-dataset cluster_dataset
  	
  	
```

- Create the GKE cost breakdown table from console google


------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------------
