

## Task 1. Set the region and zone


```bash
	gcloud config set compute/zone us-central1-c
	
	export REGION=us-central1
	
	export ZONE=us-central1-c
	
```


## Task 2. Creating a private cluster


```bash
	gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""
```


## Task 3. Viewing your subnet and secondary address ranges

```bash

	## List the subnets in the default net:
	
	gcloud compute networks subnets list --network default
	
	
	## Get info about subnet
	gcloud compute networks subnets describe gke-private-cluster-subnet-6483be1d --region $REGION
```


## Task 4. Enabling master authorized networks

```bash
	## Create a VM instance
	
	gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'
	
	## Get external IP
	gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
	
	
	## Authorize Vm External IP to access the private cluster
	gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks 34.172.40.255/32 #[MY_EXTERNAL_RANGE]
```

## Task 5. Clean Up

```bash
	gcloud container clusters delete private-cluster --zone=$ZONE	
```

## Task 6. Creating a private cluster that uses a custom subnetwork

```bash
	## Create a subnetwork and seondary ranges
	gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region=$REGION \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14
    
    
    	## Create a private cluster in custom subnet
    	
    	gcloud beta container clusters create private-cluster2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE
    
   	## Update the cluster with  Vm Ip
   	
   	gcloud container clusters update private-cluster2 \
    --enable-master-authorized-networks \
    --zone=$ZONE \
    --master-authorized-networks 34.172.40.255/32 ##[MY_EXTERNAL_RANGE] 
    
    ## Verification
    ssh vm & get cluster node info
```



------------------------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------------


