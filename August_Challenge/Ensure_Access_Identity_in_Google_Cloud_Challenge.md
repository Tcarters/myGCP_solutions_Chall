

## Task 1. Create a custom security role.


```bash
	nano role-definition.yaml

	title: "orca_storage_creator_977"
	description: "Permissions"
	stage: "ALPHA"
	includedPermissions:
	- storage.buckets.get
	- storage.objects.get
	- storage.objects.list
	- storage.objects.update
	- storage.objects.create

	## Execute file with 
	
	gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml

```


## Task 2. Create a service account.


```bash
	export SA="orca-private-cluster-558-sa"
	gcloud iam service-accounts create ${SA} --display-name "my service account"	
```

## Task 3. Bind a custom security role to a service account.

```bash
	export IAMROLE="orca_storage_creator_977"	
	
	gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SA@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.viewer

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SA@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.metricWriter

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SA@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/logging.logWriter

gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:$SA@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role projects/$DEVSHELL_PROJECT_ID/roles/orca_storage_creator_977 



```


## Task 4. Create and configure a new Kubernetes Engine private cluster

```bash
	export ZONE="us-east1-b"
	gcloud beta container clusters create orca-cluster-835 \
    --enable-private-nodes \
    --enbale-private-endpoint \
    --enable-master-authorized-networks \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork orca-build-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE

	gcloud container --project "qwiklabs-gcp-03-85b47e26a7d1" clusters create-auto "orca-cluster-835" --region "us-east1" --release-channel "regular" --enable-private-nodes --enable-private-endpoint --master-ipv4-cidr "172.16.0.0/28" --enable-master-authorized-networks --master-authorized-networks 192.168.10.2/32 --network "projects/qwiklabs-gcp-03-85b47e26a7d1/global/networks/orca-build-vpc" --subnetwork "projects/qwiklabs-gcp-03-85b47e26a7d1/regions/us-east1/subnetworks/orca-build-subnet" --cluster-ipv4-cidr "/17" --service-account $SA@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --zone $ZONE

```

- Test in Vm

```bash
	sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
	
	gcloud container clusters get-credentials orca-cluster-835 --region=us-east1
	
```

## Task 5. Deploy an application to a private Kubernetes Engine cluster.

```bash
	kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
	
```

----------------------------------------------------------------------🎉GAME_OVER🎉------------------------------------------------------------------------------
