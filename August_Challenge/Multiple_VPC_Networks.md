

## Task 1. Create custom mode VPC networks with firewall rules


```bash
    gcloud compute networks create managementnet --project=qwiklabs-gcp-02-6d3345b79bd6 --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional

    gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-02-6d3345b79bd6 --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-east1

```

### Create the privatenet network

```bash
    gcloud compute networks create privatenet --subnet-mode=custom

    ### Create privated subnet 
    gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-east1 --range=172.16.0.0/24

    ### Second privatesubnet for eu region
    gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

    ## List available net
    gcloud compute networks list

```

### Create the firewall rules for managementnet

- Create firewall rules to allow SSH, ICMP, and RDP ingress traffic to VM instances on the managementnet network.


```bash
    gcloud compute --project=qwiklabs-gcp-02-6d3345b79bd6 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,tcp:80,icmp --source-ranges=0.0.0.0/0

```

### Create the firewall rules for privatenet

- Create the firewall rules for privatenet network using the Cloud Shell command line.

```bash
    gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

    ## List firewall Rules
    gcloud compute firewall-rules list --sort-by=NETWORK

```

## Task 2. Create VM instances

### Create the managementnet-us-vm instance
- Using network `managementnet` and subnet `managementsubnet-us`

```bash
    gcloud compute instances create managementnet-us-vm --project=qwiklabs-gcp-02-6d3345b79bd6 --zone=us-east1-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=54576883464-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230711,mode=rw,size=10,type=projects/qwiklabs-gcp-02-6d3345b79bd6/zones/us-east1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any

```

### Create the privatenet-us-vm instance
- Create the privatenet-us-vm instance using the Cloud Shell command line.


```bash
    gcloud compute instances create privatenet-us-vm --zone="us-east1-c" --machine-type=e2-micro --subnet=privatesubnet-us

    ## Sort created VM
    gcloud compute instances list --sort-by=ZONE

```

## Task 3. Explore the connectivity between VM instances

### Ping the external IP addresses


```bash
    ping -c3 <ip>
```

## Task 4. Create a VM instance with multiple network interfaces

### Create the VM instance with multiple network interfaces


```bash
    gcloud compute instances create vm-appliance --project=qwiklabs-gcp-02-6d3345b79bd6 --zone=us-east1-c --machine-type=e2-standard-4 --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=privatesubnet-us --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=54576883464-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-appliance,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230711,mode=rw,size=10,type=projects/qwiklabs-gcp-02-6d3345b79bd6/zones/us-east1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any

```