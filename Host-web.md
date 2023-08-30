

```bash
gcloud compute instances stop frontend --zone=$ZONE

gcloud compute instances stop backend --zone=$ZONE

```

- Create instance Template

```bash
	gcloud compute instance-templates create fancy-fe \
    --source-instance-zone=$ZONE \
    --source-instance=frontend
    
    gcloud compute instance-templates create fancy-fe \
    --source-instance-zone=$ZONE \
    --source-instance=backend

```

- List templates
gcloud compute instance-templates list

### Create managed instance group

```bash
	gcloud compute instance-groups managed create fancy-fe-mig \
    --zone=$ZONE \
    --base-instance-name fancy-fe \
    --size 2 \
    --template fancy-fe
    
    gcloud compute instance-groups managed create fancy-be-mig \
    --zone=$ZONE \
    --base-instance-name fancy-be \
    --size 2 \
    --template fancy-be
    
    gcloud compute instance-groups set-named-ports fancy-fe-mig \
    --zone=$ZONE \
    --named-ports frontend:8080
    
    gcloud compute instance-groups set-named-ports fancy-be-mig \
    --zone=$ZONE \
    --named-ports orders:8081,products:8082
```

### Cnfigure AutoHealing

- Create a health check that repairs the instance if it returns "unhealthy" 3 consecutive times for the frontend and backend:

```bash
	gcloud compute health-checks create http fancy-fe-hc \
    --port 8080 \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
    
    gcloud compute health-checks create http fancy-be-hc \
    --port 8081 \
    --request-path=/api/orders \
    --check-interval 30s \
    --healthy-threshold 1 \
    --timeout 10s \
    --unhealthy-threshold 3
    
```

- Create a firewall rule to allow the health check probes to connect to the microservices on ports 8080-8081:

```bash
	gcloud compute firewall-rules create allow-health-check \
    --allow tcp:8080-8081 \
    --source-ranges 130.211.0.0/22,35.191.0.0/16 \
    --network default
```

- Apply health Check to services:

```bash
	gcloud compute instance-groups managed update fancy-fe-mig \
    --zone=$ZONE \
    --health-check fancy-fe-hc \
    --initial-delay 300
    
    gcloud compute instance-groups managed update fancy-be-mig \
    --zone=$ZONE \
    --health-check fancy-be-hc \
    --initial-delay 300
```


### Task 6. Create load balancers

- Create health checks that will be used to determine which instances are capable of serving traffic for each service:

```bash
	gcloud compute http-health-checks create fancy-fe-frontend-hc \
  --request-path / \
  --port 8080
  
  gcloud compute http-health-checks create fancy-be-orders-hc \
  --request-path /api/orders \
  --port 8081
  
  gcloud compute http-health-checks create fancy-be-products-hc \
  --request-path /api/products \
  --port 8082
  
```

- Create backend serivces 

```bash
	gcloud compute backend-services create fancy-fe-frontend \
  --http-health-checks fancy-fe-frontend-hc \
  --port-name frontend \
  --global
  
  	gcloud compute backend-services create fancy-be-orders \
  --http-health-checks fancy-be-orders-hc \
  --port-name orders \
  --global
  
  	gcloud compute backend-services create fancy-be-products \
  --http-health-checks fancy-be-products-hc \
  --port-name products \
  --global
  
  	
```

- Add hthe Load Balancers

```bash
	gcloud compute backend-services add-backend fancy-fe-frontend \
  --instance-group-zone=$ZONE \
  --instance-group fancy-fe-mig \
  --global
  
  gcloud compute backend-services add-backend fancy-be-orders \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global
  
  gcloud compute backend-services add-backend fancy-be-products \
  --instance-group-zone=$ZONE \
  --instance-group fancy-be-mig \
  --global

```

- Create URL map

```bash
	gcloud compute url-maps create fancy-map \
  --default-service fancy-fe-frontend
  
  
```

- Create a path matcher to allow `api/orders` `api/products`

```bash
	gcloud compute url-maps add-path-matcher fancy-map \
   --default-service fancy-fe-frontend \
   --path-matcher-name orders \
   --path-rules "/api/orders=fancy-be-orders,/api/products=fancy-be-products"
   
```

- Create proxy which ties the URL map

```bash
	gcloud compute target-http-proxies create fancy-proxy \
  --url-map fancy-map
  

```

- Create global forwarding 

```bash
	gcloud compute forwarding-rules create fancy-http-rule \
  --global \
  --target-http-proxy fancy-proxy \
  --ports 80
```

### Scaling Compute Engine

- Automatically resize by utilization

```bash
	gcloud compute instance-groups managed set-autoscaling \
  fancy-fe-mig \
  --zone=$ZONE \
  --max-num-replicas 2 \
  --target-load-balancing-utilization 0.60
```

- Enable content delivery network

```bash
	gcloud compute backend-services update fancy-fe-frontend \
    --enable-cdn --global
```

## Task 8 
