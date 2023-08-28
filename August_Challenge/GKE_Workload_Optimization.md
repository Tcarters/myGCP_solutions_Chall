
## Setup and requirements


```bash
	gcloud config set compute/zone us-central1-a
	
	gcloud container clusters create test-cluster --num-nodes=3  --enable-ip-alias
	
	gcloud container clusters get-credentials  test-cluster --zone us-central1-a --project ${GOOGLE_CLOUD_PROJECT}
```

- Create a manifest for the `gb-frontend` pod:

```bash
	cat << EOF > gb_frontend_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: gb-frontend
  name: gb-frontend
spec:
    containers:
    - name: gb-frontend
      image: gcr.io/google-samples/gb-frontend:v5
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
      ports:
      - containerPort: 80
EOF

kubectl apply -f gb_frontend_pod.yaml

```


## Task 1. Container-native load balancing through ingress

- Configure a ClusterIP:

```bash
	cat << EOF > gb_frontend_cluster_ip.yaml
apiVersion: v1
kind: Service
metadata:
  name: gb-frontend-svc
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
spec:
  type: ClusterIP
  selector:
    app: gb-frontend
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
EOF

kubectl apply -f gb_frontend_cluster_ip.yaml
```


- Create an ingress for our app

```bash
	cat << EOF > gb_frontend_ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gb-frontend-ingress
spec:
  defaultBackend:
    service:
      name: gb-frontend-svc
      port:
        number: 80
EOF

kubectl apply -f gb_frontend_ingress.yaml


```


- Check the health status

```bash
	BACKEND_SERVICE=$(gcloud compute backend-services list | grep NAME | cut -d ' ' -f2)
	
	gcloud compute backend-services get-health $BACKEND_SERVICE --global
	
	kubectl get ingress gb-frontend-ingress
	
```


## Task 2. Load testing an application

```bash
	gsutil -m cp -r gs://spls/gsp769/locust-image .
	
	## Build the Docker image for Locust
	
	gcloud builds submit \
    --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/locust-tasks:latest locust-image
    

	## Verify the Docker image 
	
	gcloud container images list
	
	
	## Apply manifest that will create a single-pod deploy
	
	gsutil cp gs://spls/gsp769/locust_deploy_v2.yaml .
	sed 's/${GOOGLE_CLOUD_PROJECT}/'$GOOGLE_CLOUD_PROJECT'/g' locust_deploy_v2.yaml | kubectl apply -f -
	
	## Access Locust UI
	kubectl get service locust-main

	
```

## Task 3. Readiness and liveness probes

```bash
	cat << EOF > liveness-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    demo: liveness-probe
  name: liveness-demo-pod
spec:
  containers:
  - name: liveness-demo-pod
    image: centos
    args:
    - /bin/sh
    - -c
    - touch /tmp/alive; sleep infinity
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/alive
      initialDelaySeconds: 5
      periodSeconds: 10
EOF


	## verify the health of the pods
	
	kubectl apply -f liveness-demo.yaml
	
	kubectl describe pod liveness-demo-pod
	
	## Delete the file being used by the liveness probe:
	
	kubectl exec liveness-demo-pod -- rm /tmp/alive
	
```


- Setting up a readiness probe

```bash
	cat << EOF > readiness-demo.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    demo: readiness-probe
  name: readiness-demo-pod
spec:
  containers:
  - name: readiness-demo-pod
    image: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthz
      initialDelaySeconds: 5
      periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: readiness-demo-svc
  labels:
    demo: readiness-probe
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  selector:
    demo: readiness-probe
EOF
```

- Apply 

```bash
	kubectl apply -f readiness-demo.yaml
	
	kubectl get service readiness-demo-svc
	
	kubectl describe pod readiness-demo-pod
	
	## Generate file for readiness probe
	
	kubectl exec readiness-demo-pod -- touch /tmp/healthz
	
	## Checking: 
	
	kubectl describe pod readiness-demo-pod | grep ^Conditions -A 5
```


## Task 4. Pod disruption budgets


```bash
	# Delete pod
	kubectl delete pod gb-frontend
	
	## Create a Deployment with 5 replicas
	
	cat << EOF > gb_frontend_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gb-frontend
  labels:
    run: gb-frontend
spec:
  replicas: 5
  selector:
    matchLabels:
      run: gb-frontend
  template:
    metadata:
      labels:
        run: gb-frontend
    spec:
      containers:
        - name: gb-frontend
          image: gcr.io/google-samples/gb-frontend:v5
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
          ports:
            - containerPort: 80
              protocol: TCP
EOF

	
		## Execute
		
		kubectl apply -f gb_frontend_deployment.yaml
		

```

- Drain the nodes by looping through the output of the default-pool's nodes and running the kubectl drain command on each individual node:

```bash

	for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
  kubectl drain --force --ignore-daemonsets --grace-period=10 "$node";
done


	## Checking deploy
	
	kubectl describe deployment gb-frontend | grep ^Replicas

```

- First bring the drained nodes back by uncordoning them. The command below allows pods to be scheduled on the node again:

```bash
	for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
  kubectl uncordon "$node";
done

	kubectl describe deployment gb-frontend | grep ^Replicas
	
	## Create a pod disruption budget
	
	kubectl create poddisruptionbudget gb-pdb --selector run=gb-frontend --min-available 4
	
```

- Result

```bash
	for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=default-pool -o=name); do
  kubectl drain --timeout=30s --ignore-daemonsets --grace-period=10 "$node";
done

kubectl describe deployment gb-frontend | grep ^Replicas
```


------------------------------------------------------🎉END-GAME🎉----------------------------------------------------------------------------
