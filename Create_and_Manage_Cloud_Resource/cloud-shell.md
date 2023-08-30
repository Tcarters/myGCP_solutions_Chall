
- Set a Region 

```bash
	gcloud config set compute/region us-east1		
```

- Get the value

```bash
	gcloud config get-value compute/region
```

- Set the Zone
```bash
	gcloud config set compute/zone us-east1-c
```

- View project zone setting
```bash
	gcloud config get-value compute/zone
```

- Get Project ID
```bash
	gcloud config get-value project
```

- View details about the project
```bash
	gcloud compute project-info describe --project $(gcloud config get-value project)
	
```

### Setting environment variables

```bash
	export PROJECT_ID=$(gcloud config get-value project)
	
	export ZONE=$(gcloud config get-value compute/zone)
	
	
	echo -e "PROJECT ID: $PROJECT_ID\nZONE: $ZONE"
	
	
```

### Creating a virtual machine with the gcloud tool

```bash
	gcloud compute instances create gcelab2 --machine-type e2-medium --zone $ZONE
```

- Filtering Output

```bash
	gcloud compute instances list --filter="name=('gcelab2')"
```

- Listing Firewalls in the project

```bash
	gcloud compute firewall-rules list
```

-List firewall-rules matching an ICMP rule

```bash
	gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'icmp'"
```


### Connect to an Instance 

```bash
	gcloud compute ssh gcelab2 --zone $ZONE
```

- Add a tag to the virtual machine:

```bash
	gcloud compute instances add-tags gcelab2 --tags http-server,https-server
	
	
	## Update firewall rules to below:
	gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
	
	## List new firewall
	gcloud compute firewall-rules list --filter=ALLOW:'80'
```

- Verify communication:

```bash
	curl http://$(gcloud compute instances list --filter=name:gcelab2 --format='value(EXTERNAL_IP)')
	
```

## Viewing system logs

- Logs on system
```bash
	gcloud logging logs list
```

- View logs related to compute resources:

```bash
	gcloud logging logs list --filter="compute"
```

- Read the logs related to the resouces type of
```bash
	gcloud logging read "resource.type=gce_instance" --limit 5
	
	gcloud logging read "resource.type=gce_instance AND labels.instance_name='gcelab2'" --limit 5
	
	
```
