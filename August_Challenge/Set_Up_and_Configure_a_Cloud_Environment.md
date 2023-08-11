# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab (GSP321)

Challenge scenario
As a cloud engineer in Jooli Inc. and recently trained with Google Cloud and Kubernetes you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

You need to complete the following tasks:

Create a development VPC with three subnets manually
Create a production VPC with three subnets manually
Create a bastion that is connected to both VPCs
Create a development Cloud SQL Instance and connect and prepare the WordPress environment
Create a Kubernetes cluster in the development VPC for WordPress
Prepare the Kubernetes cluster for the WordPress environment
Create a WordPress deployment using the supplied configuration
Enable monitoring of the cluster via stackdriver
Provide access for an additional engineer
Some Jooli Inc. standards you should follow:

Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.

Use the project VPCs.

Naming is normally team-resource, e.g. an instance could be named kraken-webserver1.

Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use e2-medium.


## Task 1. Create development VPC manually

```bash
	gcloud compute networks create griffin-dev-vpc --project=qwiklabs-gcp-03-de19045744eb --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
	
	gcloud compute networks subnets create griffin-dev-wp --project=qwiklabs-gcp-03-de19045744eb --range=192.168.16.0/20 --stack-type=IPV4_ONLY --network=griffin-dev-vpc --region=us-east1
	
	gcloud compute networks subnets create griffin-dev-mgmt --project=qwiklabs-gcp-03-de19045744eb --range=192.168.32.0/20 --stack-type=IPV4_ONLY --network=griffin-dev-vpc --region=us-east1
```


## Task 2. Create production VPC manually

```bash
	gcloud compute networks create griffin-prod-vpc --project=qwiklabs-gcp-03-de19045744eb --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
	
	gcloud compute networks subnets create griffin-prod-wp --project=qwiklabs-gcp-03-de19045744eb --range=192.168.48.0/20 --stack-type=IPV4_ONLY --network=griffin-prod-vpc --region=us-east1
	
	gcloud compute networks subnets create griffin-prod-mgmt --project=qwiklabs-gcp-03-de19045744eb --range=192.168.64.0/20 --stack-type=IPV4_ONLY --network=griffin-prod-vpc --region=us-east1
	
		
```

- Allow ssh

```bash
	### First Network `DEV VPC`
	gcloud compute --project=qwiklabs-gcp-03-de19045744eb firewall-rules create allow-ssh-dev --direction=INGRESS --priority=1000 --network=griffin-dev-vpc --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0 --target-tags=ssh
	
	## Second network `PRODUCTION VPC`
	gcloud compute --project=qwiklabs-gcp-03-de19045744eb firewall-rules create allow-ssh-on-prod --direction=INGRESS --priority=1000 --network=griffin-prod-vpc --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0 --target-tags=ssh
```


## Task 3. Create bastion host

```bash
	gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b

```

## Task 4. Create and configure Cloud SQL Instance


```bash
	gcloud sql instances create griffin-dev-db --root-password 	superSecret@2023 --region=us-east1


```

## Task 5. Create Kubernetes cluster

```bash
	gcloud container clusters create griffin-dev --machine-type=e2-standard-4 --zone=us-east1-b --network griffin-dev-vpc --subnetwork griffin-dev-wp --num-nodes 2
	
		gcloud container clusters get-credentials griffin-dev --zone us-east1-b
```

## Task 6. Prepare the Kubernetes cluster

```bash
	gsutil cp -r gs://cloud-training/gsp321/wp-k8s .

	cd wp-k8s

	sed -i s/username_goes_here/wp_user/g wp-env.yaml

	sed -i s/password_goes_here/stormwind_rules/g wp-env.yaml
	
	## Create k8s Key for deployment
	gcloud iam service-accounts keys create key.json \
    		--iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

	kubectl create secret generic cloudsql-instance-credentials \
	    --from-file key.json
    
    
```

## Task 7. Create a WordPress deployment


```bash
	export INSTANCE_NAME=$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")
	
	## INSTANCE_HOSTNAME should look like this on UI: qwiklabs-gcp-03-de19045744eb:us-east1:griffin-dev-db
	
	sed -i 's/YOUR_SQL_INSTANCE/$INSTANCE_NAME/g' wp-deployment.yaml

	kubectl create -f wp-deployment.yaml

	kubectl create -f wp-service.yaml

```

- COntent of `w--deployment.yml`

```yaml
	apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
        - image: wordpress
          name: wordpress
          env:
          - name: WORDPRESS_DB_HOST
            value: 127.0.0.1:3306
          - name: WORDPRESS_DB_USER
            valueFrom:
              secretKeyRef:
                name: database
                key: username
          - name: WORDPRESS_DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: database
                key: password
          ports:
            - containerPort: 80
              name: wordpress
          volumeMounts:
            - name: wordpress-persistent-storage
              mountPath: /var/www/html
        - name: cloudsql-proxy
          image: gcr.io/cloudsql-docker/gce-proxy:1.33.2
          command: ["/cloud_sql_proxy",
                    "-instances=qwiklabs-gcp-03-de19045744eb:us-east1:griffin-dev-db=tcp:3306",
                    "-credential_file=/secrets/cloudsql/key.json"]
          securityContext:
            runAsUser: 2  # non-root user
            allowPrivilegeEscalation: false
          volumeMounts:
            - name: cloudsql-instance-credentials
              mountPath: /secrets/cloudsql
              readOnly: true
      volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wordpress-volumeclaim
        - name: cloudsql-instance-credentials
          secret:
            secretName: cloudsql-instance-credentials
```

- Content of `wp-service.yml`

```yaml
	apiVersion: v1
kind: Service
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app: wordpress
```

- Content of `wp-env.yaml`

```yaml
	kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wordpress-volumeclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: database
type: Opaque
stringData:
  username: wp_user
  password: stormwind_rules
```


## Task 8. Enable monitoring

- Get Endpoint IP by going to:

> Navigation Menu -> Kubernetes Engine -> Services and Ingress -> Copy Endpoint's address.

> or in `Coud Shell` , type : `kubectl get services`

- Going to `OPERATIONS` `Monitoring` > `Uptime Checks`:
> + CREATE UPTIME CHECK Title : Wordpress Uptime ,
> Next -> Target Hostname : {Endpoint's address} (without http...) ,
> and For Path : / 
> Next -> Next -> Create


## Task 9. Provide access for an additional engineer
- I assume you can do it very easy ))..

- Go to the Navigation Menu: `IAM & Admin` >
+ IAM -> next
+ Get the Username 2 from Lab instruction page and select edit access: 
+ Search Role : `Project` -> and select  `Editor` 
+ Save...
