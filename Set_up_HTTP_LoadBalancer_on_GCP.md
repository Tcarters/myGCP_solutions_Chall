# Process to setup an HTTP load Balancer 


## Requirements
- Set up Zone and Regions

```bash
	gcloud config set compute/region us-west3
	export region=us-west3
	
	gcloud config set compute/zone us-west3-b
	export zone=us-west3-b
```

- - -

## Create an Instance 

```bash
	  gcloud compute instances create web-app \
    --zone=$zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
	apt-get update
	apt-get install -y nginx
	service nginx start
	sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html'
      
```

- Allow firewall rule

```bash
	gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
    
```
- - -


## Phase1: Create an Instance Template  from source instance
	--source-instance=web-app \
	
```bash
	gcloud compute instance-templates create nucleus-web-template \
	--region=$region \
   	--network=nucleus-vpc \
   	--tags=nucleus-web \
   	--machine-type=e2-small \
   	--image-family=debian-11 \
   	--image-project=debian-cloud \
   	--metadata=startup-script='#! /bin/bash
	   	apt-get update
	   	apt-get install -y nginx
	   	service nginx start
		sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"' /' /var/www/html/index.nginx-debian.html'

```
## Phase2: Create a Managed Instance Group

```bash
	gcloud compute instance-groups managed create nucleus-web-mig \
   --template=nucleus-web-template --size=2 --region=$region --base-instance-name web-app

```


## Create a Target Pool

```bash
	  gcloud compute target-pools create nucleus-pool \
    --region $region --http-health-check http-basic-check
```

## Phase3: Create a firewall rule to allow traffic (80/tcp)

```bash

gcloud compute firewall-rules create allow-tcp-rule-148 \
	--network=nucleus-vpc \
  	--allow tcp:80 \
  	--direction=ingress \
  	--target-tags=nucleus-web
  
  OR
  gcloud compute firewall-rules create allow-tcp-health-check \
  --network=nucleus-vpc \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16,0.0.0.0/0 \
  --target-tags=nucleus-web \
  --rules=tcp:80
```


## Phase4: Create a Health Check

```bash
   gcloud compute http-health-checks create http-basic-check 

```

## Phase5: Set named-port to insatnces group

```bash
	gcloud compute instance-groups managed \
          set-named-ports nucleus-web-mig \
          --named-ports http:80 \
          --region $region

```

## Phase6: Create a Backend service 

```bash
	gcloud compute backend-services create nucleus-backend-service \
  --protocol=HTTP \
  --http-health-checks=http-basic-check \
  --global
  
```

## Phase7: Add your instance group as the backend to the backend service:

```bash
	gcloud compute backend-services add-backend nucleus-backend-service \
  --instance-group=nucleus-web-mig \
  --instance-group-region=$region \
  --global
```

## Phase8: Create a URL Map

```bash
	gcloud compute url-maps create nucleus-map-http \
    --default-service nucleus-backend-service
```

## Phase9: Create a Target HTTP proxy to route requests to URL map

```bash
	gcloud compute target-http-proxies create http-nucleus-proxy \
    --url-map nucleus-map-http
```
	
## Phase10: Create a global forwarding rule to route incoming requests to the proxy

```bash
	gcloud compute forwarding-rules create http-content-rule \
    --global \
    --target-http-proxy=http-nucleus-proxy \
    --ports=80
```
## Phase11: List forwarding rule

```bash
	gcloud compute forwarding-rules list
```
    
