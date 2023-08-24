


## Task 1. VPC network peering setup


### Create a custom network in projects : 

#### Project A setup

- Create custom network

```bash
    gcloud compute networks create network-a --subnet-mode custom

```

- Create a subnet within VPC

```bash
    gcloud compute networks subnets create network-a-subnet --network network-a \
    --range 10.0.0.0/16 --region us-west1 

```

- Create a VM instance:

`gcloud compute instances create vm-a --zone us-west1-a --network network-a --subnet network-a-subnet --machine-type e2-small `

- Enable SSH and icmp 

```bash
    gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp 
```

```bash

    gcloud compute instances create vm-a --zone us-west1-a --network network-a --subnet network-a-subnet --machine-type e2-small 

```

#### Project B setup

- Same process

```bash
	gcloud compute networks create network-b --subnet-mode custom
	
	gcloud compute networks subnets create network-b-subnet --network network-b \
    --range 10.8.0.0/16 --region us-west1 
    
    	gcloud compute instances create vm-b --zone us-west1-a --network network-b --subnet network-b-subnet --machine-type e2-small
    	
    	gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp
```


## Task 2. Setting up a VPC Network Peering session


 Confere Console
 
 `gcloud compute routes list --project qwiklabs-gcp-02-b0de1d501876 `
 
 
## Task 3. Connectivity test

- SSH Vm-b and ping VM-a


--------------------------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------
