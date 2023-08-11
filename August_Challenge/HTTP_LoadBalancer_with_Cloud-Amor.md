
## Task 1. Configure HTTP and health check firewall rules


### Create the HTTP firewall rule


```bash
    gcloud compute --project=qwiklabs-gcp-01-a1a270a7c9c5 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

### Create the health check firewall rules

```bash
    gcloud compute --project=qwiklabs-gcp-01-a1a270a7c9c5 firewall-rules create default-allow-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=http-server
```

## Task 2. Configure instance templates and create instance groups

### Configure the instance templates


```bash
    gcloud compute instance-templates create us-west3-template --project=qwiklabs-gcp-01-a1a270a7c9c5 --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh$'\n',enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=1031917462957-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-west3 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-west3-template,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230711,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any


```

- Second template of same nature but with subnetwork in europe-west1

```bash
    gcloud compute instance-templates create europe-west1-template --project=qwiklabs-gcp-01-a1a270a7c9c5 --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh$'\n',enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=1031917462957-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=europe-west1 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-west3-template,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230711,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

```

## Create the managed instance groups

```bash
    gcloud beta compute instance-groups managed create us-west3-mig --project=qwiklabs-gcp-01-a1a270a7c9c5 --base-instance-name=us-west3-mig --size=1 --template=us-west3-template --zones=us-west3-a,us-west3-b,us-west3-c --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --list-managed-instances-results=PAGELESS --no-force-update-on-repair

    ## Set autoscaling
    gcloud beta compute instance-groups managed set-autoscaling us-west3-mig --project=qwiklabs-gcp-01-a1a270a7c9c5 --region=us-west3 --cool-down-period=45 --max-num-replicas=2 --min-num-replicas=1 --mode=on --target-cpu-utilization=0.8


```

- Second Instance group for region europe-west1

```bash
    gcloud beta compute instance-groups managed create europe-west1-mig --project=qwiklabs-gcp-01-a1a270a7c9c5 --base-instance-name=europe-west1-mig --size=1 --template=europe-west1-template --zones=europe-west1-b,europe-west1-d,europe-west1-c --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --list-managed-instances-results=PAGELESS --no-force-update-on-repair

    gcloud beta compute instance-groups managed set-autoscaling europe-west1-mig --project=qwiklabs-gcp-01-a1a270a7c9c5 --region=europe-west1 --cool-down-period=45 --max-num-replicas=2 --min-num-replicas=1 --mode=on --target-cpu-utilization=0.8
```

## Task 3. Configure the HTTP Load Balancer

- Configure the HTTP Load Balancer to balance traffic between the two backends (us-east1-mig in us-east1 and europe-west1-mig in europe-west1)


```bash

```