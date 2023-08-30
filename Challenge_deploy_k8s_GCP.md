# Publish to GCp Artifact Repository

### Create Docker Image and store

- Can use this cmd to clone repo

```bash
	gcloud source repos clone valkyrie-app
```

- Create Dockerfile

```Dockerfile
	FROM golang:1.10
	WORKDIR /go/src/app
	COPY source .
	RUN go install -v
	ENTRYPOINT ["app","-single=true","-port=8080"]
	
```

- create Docker image

```bash
	cd valkyrie-app 
	docker build -t valkyrie-app:v0.0.2 .
```

- Launch a container

```bash
	 docker run -dit -p 8080:8080 valkyrie-app:v0.0.2
```

## Task Push Docker image to Artifact Registry :

### Configure Authentication

```bash
	gcloud auth configure-docker us-central1-docker.pkg.dev
```

### Build or Tag an Image

- Build with :
```bash
	docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2 .
```

- Tag with:

```bash
	docker tag valkyrie-app:v0.0.2  us-central1-docker.pkg.dev/qwiklabs-gcp-01-c65458000db3/valkyrie-docker-repo/valkyrie-app:v0.0.2
	
```

### Push Docker image to Artifact Registry

- Push with :
```bash
	docker push us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
	
	OR for another project
	
	docker push us-central1-docker.pkg.dev/qwiklabs-gcp-01-c65458000db3/valkyrie-docker-repo/valkyrie-app:v0.0.2
	 
```



##  Create and expose a deployment in Kubernetes

- Get Auth of cluster running in GKE

```bash
	gcloud container clusters get-credentials < cluster-name >
	
	#in this case
	gcloud container clusters get-credentials valkyrie-dev --zone=us-east1-d
```

- Create deployment

```yaml
	kind: Deployment
apiVersion: apps/v1
metadata:
  name: valkyrie-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: valkyrie-app
  template:
    metadata:
      name: valkyrie-app
      labels:
        app: valkyrie-app
        role: frontend-backend
        env: dev
    spec:
      containers:
      - name: backend
        image: us-central1-docker.pkg.dev/qwiklabs-gcp-01-c65458000db3/valkyrie-docker-repo/valkyrie-app:v0.0.2
        resources:
          limits:
            memory: "500Mi"
            cpu: "100m"
        imagePullPolicy: Always
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
        command: ["app","-port=8080"]
        ports:
        - name: backend
          containerPort: 8080
      - name: frontend
        image: us-central1-docker.pkg.dev/qwiklabs-gcp-01-c65458000db3/valkyrie-docker-repo/valkyrie-app:v0.0.2
        resources:
          limits:
            memory: "500Mi"
            cpu: "100m"
        imagePullPolicy: Always
        readinessProbe:
          httpGet:
            path: /healthz
            port: 80
        command: ["app","-frontend=true","-backend-service=http://127.0.0.1:8080","-port=80"]
        ports:
        - name: frontend
          containerPort: 80
```

- Lauch deployment :
```bash
	cd /home/student_02_e8aa46591291/marking/valkyrie-app/k8s
	
	kubectl create -f deployment.yaml 
```

- Launch service

```yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app: valkyrie-dev
  name: valkyrie-dev
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: valkyrie-app
    role: frontend-backend
    env: dev
  sessionAffinity: None
  type: LoadBalancer
```

- Launch service :
 
```bash
	kubectl create -f service.yaml
```

- Verification

```bash
	watch -n2 kubectl get service
```
