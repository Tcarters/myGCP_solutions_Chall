gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID

 ----------
 
Task 3. Create a new instance with gcloud

- gcloud compute instances create gcelab2 --machine-type e2-medium --zone=$ZONE

- Connect to Instance via Gcloud:

> gcloud compute ssh gcelab2 --zone=$ZONE

## Host Hosting a Web App on Google Cloud Using Compute Engine

### Set the region and zone

1. Set the project region for this lab:
- gcloud config set compute/region us-central1

2. Create a variable for region:

- export REGION=us-central1

3. Create a variable for zone:

- export ZONE=us-central1-c


### Task 1. Enable Compute Engine API

- gcloud services enable compute.googleapis.com


### Task 2. Create Cloud Storage bucket

- gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID

### Task 3. Clone source repository

1. Clone the source code and then navigate to the monolith-to-microservices directory:

- git clone https://github.com/googlecodelabs/monolith-to-microservices.git

- cd ~/monolith-to-microservices

2. Run the initial build of the code to allow the application to run locally:

- ./setup.sh

- Once completed, ensure Cloud Shell is running a compatible nodeJS version with the following command:

nvm install --lts

4. Next, run the following to test the application, switch to the microservices directory, and start the web server:

cd microservices
npm start




------------
### Task 4. Create Compute Engine instances :

1. Create a virtual machine www1 in your default zone using the following code:

  gcloud compute instances create www2 \
    --zone=us-east4-c \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www2</h3>" | tee /var/www/html/index.html'


2. Create a firewall rule to allow external traffic to the VM instances:

gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
    

3. List instances:
gcloud compute instances list


### Task 5. Configure the load balancing service

1.   Create a static external IP address for your load balancer:

 gcloud compute addresses create network-lb-ip-1 \
    --region us-east4
	

2. Add a legacy HTTP health check resource:

gcloud compute http-health-checks create basic-check


3. Add a target pool in the same region as your instances. Run the following to create the target pool and use the health check, which is required for the service to function:

  gcloud compute target-pools create www-pool \
    --region us-east4 --http-health-check basic-check

4. Add the instances to the pool:

gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3
    
5. Add a forwarding rule:

gcloud compute forwarding-rules create www-rule \
    --region  us-east4 \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool


### Task 4. Sending traffic to your instances

1. Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer:

gcloud compute forwarding-rules describe www-rule --region us-east4


2. Access the external IP address

IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-east4 --format="json" | jq -r .IPAddress)

3. Use curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command:

while true; do curl -m1 $IPADDRESS; done


### Task 5. Create an HTTP load balancer
To set up a load balancer with a Compute Engine backend, your VMs need to be in an instance group. The managed instance group provides VMs running the backend servers of an external HTTP load balancer. For this lab, backends serve their own hostnames.

1. First, create the load balancer template:

gcloud compute instance-templates create lb-backend-template \
   --region=us-east4 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --machine-type=e2-medium \
   --image-family=debian-11 \
   --image-project=debian-cloud \
   --metadata=startup-script='#!/bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
     
2. Create a managed instance group based on the template:

gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-east4-c 
   
   
3. Create the fw-allow-health-check firewall rule.

gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
  

4. Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer:

gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
  
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
  

5. Create a health check for the load balancer:

gcloud compute health-checks create http http-basic-check \
  --port 80

6. Create a backend service:

gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
  

7. Add your instance group as the backend to the backend service:

gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=us-east4-c \
  --global
  
8. Create a URL map to route the incoming requests to the default backend service:

gcloud compute url-maps create web-map-http \
    --default-service web-backend-service


9. Create a target HTTP proxy to route requests to your URL map:

gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
    
10. Create a global forwarding rule to route incoming requests to the proxy:

gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80


### Task 6. Testing traffic sent to your instances
In the Google Cloud console, from the Navigation menu, go to Network services > Load balancing.

Click on the load balancer that you just created (web-map-http).

In the Backend section, click on the name of the backend and confirm that the VMs are Healthy. If they are not healthy, wait a few moments and try reloading the page.

When the VMs are healthy, test the load balancer using a web browser, going to http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's IP address.

This may take three to five minutes. If you do not connect, wait a minute, and then reload the browser.

Your browser should render a page with content showing the name of the instance that served the page, along with its zone (for example, Page served from: lb-backend-group-xxxx).



- - ------------------------------------------------------------------------------------------------------------------------------------------

# INTERNAL LOADBALANCER

## Set the Region

gcloud config set compute/region us-west1 && gcloud config set compute/zone us-west1-a && export REGION=us-west1 && export ZONE=us-west1-a

### Task 1. Create a virtual environment

1. Install the virtualenv environment:


```bash
	sudo apt-get install -y virtualenv
	
	python3 -m venv venv
	
	source venv/bin/activate
```

### Task 2. Create Backend Managed Instance Group

Create the startup script which includes a simple web server in Python that will return True or False depending on if a number is prime or not.


1. Start by creating your backend.sh script in the home directory:

```bash
	touch ~/backend.sh

```
2. Content ``Backend.sh``

```Text
	sudo chmod -R 777 /usr/local/sbin/
sudo cat << EOF > /usr/local/sbin/serveprimes.py
import http.server
def is_prime(a): return a!=1 and all(a % i for i in range(2,int(a**0.5)+1))
class myHandler(http.server.BaseHTTPRequestHandler):
  def do_GET(s):
    s.send_response(200)
    s.send_header("Content-type", "text/plain")
    s.end_headers()
    s.wfile.write(bytes(str(is_prime(int(s.path[1:]))).encode('utf-8')))
http.server.HTTPServer(("",80),myHandler).serve_forever()
EOF
nohup python3 /usr/local/sbin/serveprimes.py >/dev/null 2>&1 &

```
3. create the instance template for a simple instance with no external IP address:

```Bash
	gcloud compute instance-templates create primecalc \
--metadata-from-file startup-script=backend.sh \
--no-address --tags backend --machine-type=e2-medium
```

4. Now open the firewall to port 80:

```Bash
	gcloud compute firewall-rules create http --network default --allow=tcp:80 \
--source-ranges 10.138.0.0/20 --target-tags backend

```

5. Next create the instance group:

```bash
	gcloud compute instance-groups managed create backend \
--size 3 \
--template primecalc \
--zone $ZONE

```

6. And set it to auto scale based on the CPU:

```bash
	gcloud compute instance-groups managed set-autoscaling backend \
--target-cpu-utilization 0.8 --min-num-replicas 3 \
--max-num-replicas 10 --zone $ZONE

```

### Task 3. Setup internal load balancer
Now set up the Internal Load Balancer and link it with the instance group created earlier.

An internal load balancer consists of a:

Forwarding rule which binds an internal IP address.
Backend service (which is regional) linking to one or more backend instance groups (which are zonal).
Health check attached to the backend service that determines which instances can receive new connections.

A health check is needed to balance between healthy instances only.


1. Since the HTTP service is provided, see if a 200 response on a specific URL path (in this case /2 to check if 2 is prime) is populated:

gcloud compute health-checks create http ilb-health --request-path /2


2. Create a backend service:

```bash
	gcloud compute backend-services create prime-service \
--load-balancing-scheme internal --region=$REGION \
--protocol tcp --health-checks ilb-health

```

3. Add the instance group:

```bash
	gcloud compute backend-services add-backend prime-service \
--instance-group backend --instance-group-zone=$ZONE \
--region=$REGION

```

4. Now create the forwarding rule to finalize the load balancer setup with a static IP of 10.138.10.10:

```bash
	gcloud compute forwarding-rules create prime-lb \
--load-balancing-scheme internal \
--ports 80 --network default \
--region=$REGION --address 10.138.10.10 \
--backend-service prime-service

```

### Task 4. Test the load balancer

To test the load balancer, you need to create a new instance in the same network as the internal load balancer and SSH into it. That is because the cloud shell lives outside the project network you created.

1. Using gcloud in Cloud Shell, create the new instance:

```bash
	gcloud compute instances create testinstance \
--machine-type=e2-standard-2 --zone $ZONE

```

2. SSH into IT:

```bash
	gcloud compute ssh testinstance --zone $ZONE
```
3. Check if a few numbers are prime by querying the IP of the load balancer with the curl command:

- curl 10.138.10.10/2
- curl 10.138.10.10/4
- curl 10.138.10.10/5

4. Delete it :
gcloud compute instances delete testinstance --zone=$ZONE


### Task 5. Create a public facing web server

1. Start by creating your frontend.sh script in the home directory:



```Text 
	sudo chmod -R 777 /usr/local/sbin/
sudo cat << EOF > /usr/local/sbin/getprimes.py
import urllib.request
from multiprocessing.dummy import Pool as ThreadPool
import http.server
PREFIX="http://10.138.10.10/" #HTTP Load Balancer
def get_url(number):
    return urllib.request.urlopen(PREFIX+str(number)).read().decode('utf-8')
class myHandler(http.server.BaseHTTPRequestHandler):
  def do_GET(s):
    s.send_response(200)
    s.send_header("Content-type", "text/html")
    s.end_headers()
    i = int(s.path[1:]) if (len(s.path)>1) else 1
    s.wfile.write("<html><body><table>".encode('utf-8'))
    pool = ThreadPool(10)
    results = pool.map(get_url,range(i,i+100))
    for x in range(0,100):
      if not (x % 10): s.wfile.write("<tr>".encode('utf-8'))
      if results[x]=="True":
        s.wfile.write("<td bgcolor='#00ff00'>".encode('utf-8'))
      else:
        s.wfile.write("<td bgcolor='#ff0000'>".encode('utf-8'))
      s.wfile.write(str(x+i).encode('utf-8')+"</td> ".encode('utf-8'))
      if not ((x+1) % 10): s.wfile.write("</tr>".encode('utf-8'))
    s.wfile.write("</table></body></html>".encode('utf-8'))
http.server.HTTPServer(("",80),myHandler).serve_forever()
EOF
nohup python3 /usr/local/sbin/getprimes.py >/dev/null 2>&1 &

```

2. In Cloud Shell create an instance running this web server:



```bash
	gcloud compute instances create frontend --zone=$ZONE \
--metadata-from-file startup-script=frontend.sh \
--tags frontend --machine-type=e2-standard-2
```

3. Open the firewall for the front end:

gcloud compute firewall-rules create http2 --network default --allow=tcp:80 \
--source-ranges 0.0.0.0/0 --target-tags frontend


4. Open the External IP for the frontend in your browser:

You should see a matrix like this, showing all prime numbers, up to 100, in green:




- - - - -- - - -- - ---------------------------------------------------------

## Task 4. Create custom network with Cloud Shell

1. Create custom Network

gcloud compute networks create taw-custom-network --subnet-mode custom


2. Now create subnets for your new custom network. You'll create three of them.

gcloud compute networks subnets create subnet-us-central \
   --network taw-custom-network \
   --region us-central1 \
   --range 10.0.0.0/16
   

3. Create subnet-europe-west with an IP prefix:

gcloud compute networks subnets create subnet-europe-west \
   --network taw-custom-network \
   --region europe-west1 \
   --range 10.1.0.0/16
   

4. Create subnet-asia-east with an IP prefix:

gcloud compute networks subnets create subnet-asia-east \
   --network taw-custom-network \
   --region asia-east1 \
   --range 10.2.0.0/16
   
5. List your networks:

gcloud compute networks subnets list \
   --network taw-custom-network
 
 

### Add firewall rules using Cloud Shell

  
6. To create firewall rules in Cloud Shell, run the following:

gcloud compute firewall-rules create nw101-allow-http \
--allow tcp:80 --network taw-custom-network --source-ranges 0.0.0.0/0 \
--target-tags http


- Addition Rule for ``ICMP``

```bash
	gcloud compute firewall-rules create "nw101-allow-icmp" --allow icmp --network "taw-custom-network" --target-tags rules
```

- Additional Rule for `` Internal Communication ``

```bash
	gcloud compute firewall-rules create "nw101-allow-internal" --allow tcp:0-65535,udp:0-65535,icmp --network "taw-custom-network" --source-ranges "10.0.0.0/16","10.2.0.0/16","10.1.0.0/16"
```

- For `` SSH ``


```bash
	gcloud compute firewall-rules create "nw101-allow-ssh" --allow tcp:22 --network "taw-custom-network" --target-tags "ssh"
```

- For `RDP`

```bash
	gcloud compute firewall-rules create "nw101-allow-rdp" --allow tcp:3389 --network "taw-custom-network"
```


----------------------
gcloud config set compute/zone "us-east4-c" &&
gcloud config set compute/region "us-east4" &&
export REGION="us-east4" &&
export ZONE="us-east4-c"

gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID


---------------------------------------------------------------
### Create a service account




- - -

## Orchestrating the Cloud with Kubernetes


1. Set the Zone

gcloud config set compute/zone us-central1-b

gcloud container clusters create io


2. Get the sample code :

- gsutil cp -r gs://spls/gsp021/* .

- cd orchestrate-with-kubernetes/kubernetes && ls


3. Use it to launch a single instance of the nginx container:

- kubectl create deployment nginx --image=nginx:1.10.0

4. Use the kubectl get pods command to view the running nginx container:

- kubectl get po


5. WExpose Deploym
- kubectl expose deployment nginx --port 80 --type LoadBalancer

6. List our services:

- kubectl get services


#### Creating Pods

- cd ~/orchestrate-with-kubernetes/kubernetes

- cat pods/monolith.yaml

```yaml
	apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
          
```

7. Create the monolith pod using kubectl:

- kubectl create -f pods/monolith.yaml

- kubectl get pods

- kubectl describe pods monolith


### Interacting with pods

By default, pods are allocated a private IP address and cannot be reached outside of the cluster. Use the kubectl port-forward command to map a local port to a port inside the monolith pod.


8. In the 2nd terminal, run this command to set up port-forwarding:


- kubectl port-forward monolith 10080:80

- curl http://127.0.0.1:10080

- TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')

- Access App with: curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure

- View logs: kubectl logs monolith

- Use the kubectl exec command to run an interactive shell inside the Monolith Pod. This can come in handy when you want to troubleshoot from within a container:

kubectl exec monolith --stdin --tty -c monolith -- /bin/sh


### SERVICES :

Services use labels to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster,
NodePort gives each node in the cluster an externally accessible IP and
LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.
Now you'll learn how

- cd ~/orchestrate-with-kubernetes/kubernetes

- cat pods/secure-monolith.yaml
```YAML
	apiVersion: v1
kind: Pod
metadata:
  name: "secure-monolith"
  labels:
    app: monolith
spec:
  containers:
    - name: nginx
      image: "nginx:1.9.14"
      lifecycle:
        preStop:
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
      volumeMounts:
        - name: "nginx-proxy-conf"
          mountPath: "/etc/nginx/conf.d"
        - name: "tls-certs"
          mountPath: "/etc/tls"
    - name: monolith
      image: "kelseyhightower/monolith:1.0.0"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
      livenessProbe:
        httpGet:
          path: /healthz
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        periodSeconds: 15
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        timeoutSeconds: 1
  volumes:
    - name: "tls-certs"
      secret:
        secretName: "tls-certs"
    - name: "nginx-proxy-conf"
      configMap:
        name: "nginx-proxy-conf"
        items:
          - key: "proxy.conf"
            path: "proxy.conf"
```

- Create the secure-monolith pods and their configuration data:

```bash
	kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
```

- Explore the monolith service configuration file:

cat services/monolith.yaml
```text
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```
- kubectl create -f services/monolith.yaml

- Use the gcloud compute firewall-rules command to allow traffic to the monolith service on the exposed nodeport:

```bash
	gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
```
  
### Task 8. Adding labels to pods

- kubectl get pods -l "app=monolith"

Use the kubectl label command to add the missing secure=enabled label to the secure-monolith Pod. Afterwards, you can check and see that your labels have been updated.

- kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels

Now that your pods are correctly labeled, view the list of endpoints on the monolith service:

- kubectl describe services monolith | grep Endpoints


### Task 9. Deploying applications with Kubernetes
The main benefit of Deployments is in abstracting away the low level details of managing Pods. Behind the scenes Deployments use Replica Sets to manage starting and stopping the Pods. If Pods need to be updated or scaled, the Deployment will handle that. Deployment also handles restarting Pods if they happen to go down for some reason.


1.  Get started by examining the auth deployment configuration file.


- cat deployments/auth.yaml

- kubectl create -f deployments/auth.yaml
- kubectl create -f services/auth.yaml
-kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

- kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
