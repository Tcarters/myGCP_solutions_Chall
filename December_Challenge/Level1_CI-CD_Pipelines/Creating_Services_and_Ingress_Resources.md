# Creating Services and Ingress Resources

Observe Kubernetes DNS in action.
Define various service types (ClusterIP, NodePort, LoadBalancer) in manifests along with label selectors to connect to existing labeled Pods and deployments, deploy those to a cluster, and test connectivity.
Deploy an Ingress resource that connects clients to two different services based on the URL path entered.
Verify Google Cloud network load balancer creation for type=LoadBalancer services.


## Task 1. Connect to the lab GKE cluster and test DNS

```bash
export my_zone=us-central1-b
export my_cluster=standard-cluster-1

source <(kubectl completion bash)

gcloud container clusters get-credentials $my_cluster --zone $my_zone
```

## Task 2. Create Pods and services to test DNS resolution

```bash

git clone https://github.com/GoogleCloudPlatform/training-data-analyst
ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
cd ~/ak8s/GKE_Services/

# Create service & pods
kubectl apply -f dns-demo.yaml

# Verify
kubectl get pods
```
- To get inside the cluster, open an interactive session to bash running from dns-demo-1:

```bash
kubectl exec -it dns-demo-1 -- /bin/bash

# Install tool inside pods
apt-get update
apt-get install -y iputils-ping

# Ping dns-demo-2:
ping dns-demo.default.svc.cluster.local
```
![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/a4ad0cd6-0a2d-44a2-9c8b-6b05067d8612)

## Task 3. Deploy a sample workload and a ClusterIP service

- Deploy a sample web application to your Kubernetes Engine cluster

```bash

## In the second Cloud Shell tab, change to the directory that contains the sample files for this lab:
kubectl create -f hello-v1.yaml

kubectl get deployments

```

- Define service types in the manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  type: ClusterIP
  selector:
    name: hello-v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```
- Execute : `kubectl apply -f ./hello-svc.yaml` & gte service : `kubectl get service hello-svc`

- ![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/083eac4e-e254-4f39-9e45-0df2568f1ee1)

### Test your application

- In first pod

```bash
apt-get install -y curl

curl hello-svc.default.svc.cluster.local
```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/1d96f42c-fbc3-44b7-9d81-442829819844)


## Task 4. Convert the service to use NodePort

```
# In second shell

kubectl apply -f ./hello-nodeport-svc.yaml

kubectl get service hello-svc

```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/fcced49e-4a62-472f-a3c7-f89f7bed2400)

## Task 5. Create static public IP addresses using Google Cloud Networking

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/98df1e23-48ac-4262-bceb-f4ca0e3d4973)

```
gcloud compute addresses create regional-loadbalancer --project=qwiklabs-gcp-03-d71995f581e6 --region=us-central1

```
- Reserve a second ip as global

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/804c4bdc-0489-4ca1-8ac5-69467d3393ef)

## Task 6. Deploy a new set of Pods and a LoadBalancer service

- Still on the second shell

```bash
kubectl create -f hello-v2.yaml

```
![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/38f40b10-4c3e-4a90-94f3-c046187d748f)

- Define service types in the manifest

 deploy a LoadBalancer Service using the hello-lb-svc.yaml sample manifest that has already been created for you.

```bash
export STATIC_LB=$(gcloud compute addresses describe regional-loadbalancer --region us-central1 --format json | jq -r '.address')

sed -i "s/10\.10\.10\.10/$STATIC_LB/g" hello-lb-svc.yaml

## Apply new config:
kubectl apply -f ./hello-lb-svc.yaml

```

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/8cf5f4a7-7397-4586-b8e7-47467f389fc6)

- curl [external_IP]

![image](https://github.com/Tcarters/myGCP_solutions_Chall/assets/71230412/bb088e92-391f-4478-bb6d-99106a693b6e)


## Task 7. Deploy an ingress resource

> You have two services in your cluster for the hello application. One service is hosting version 1.0 via a NodePort service, while the other service is hosting version 2.0 via a LoadBalancer service. You will now deploy an Ingress resource that will direct traffic to both services based on the URL entered by the user.

- Create an ingress resource

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.global-static-ip-name: "global-ingress"
spec:
  rules:
  - http:
      paths:
      - path: /v1
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-svc
            port:
              number: 80
      - path: /v2
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-lb-svc
            port:
              number: 80
```

```bash
## Create service

kubectl apply -f hello-ingress.yaml

kubectl describe ingress hello-ingress
curl http://[external_IP]/v1
curl http://[external_IP]/v2

```

Done...
