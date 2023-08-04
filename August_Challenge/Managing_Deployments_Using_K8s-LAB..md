
- Create a cluster with 3 nodes

```bash
	gcloud container clusters create bootcamp \
  --machine-type e2-small \
  --num-nodes 3 \
  --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
  
```

## Task 1. Learn about the deployment object

- The explain command in kubectl can tell us about the deployment object:

```bash
	kubectl explain deployment
	
	kubectl explain deployment --recursive

	kubectl explain deployment.metadata.name	
```

- Get Url

```bash
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
	
```


## Scale a Deployment

- Get explanation
```bash
	kubectl explain deployment.spec.replicas
```

- Scale a deployment

```bash
	## TO scale up
	kubectl scale deployment hello --replicas=5

	## To scale back
	kubectl scale deployment hello --replicas=3
```

- Verify that there are now 5 hello Pods running:

```bash
	kubectl get pods | grep hello- | wc -l
```


## Task 3: Rolling update

- Update a deployment

```bash
	kubectl edit deployment hello	
```

- See rollout history

```bash
	kubectl rollout history deployment/hello
```

### Pause a rolling update

- Pause a rollout deploy
```bash
	kubectl rollout pause deployment/hello
```

- Verify current status of the rollout

```bash
	kubectl rollout status deployment/hello
	
	kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
	
```


### Resume a rolling update

```bash
	kubectl rollout resume deployment/hello
```

### Rollback an update

```bash
	kubectl rollout undo deployment/hello
```

- Verify the roll back 

```bash
	kubectl rollout history deployment/hello
```


## Task 4: canary deployments

When you want to test a new deployment in production with a subset of your users, use a canary deployment. Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.

### Create a canary deployment

- A canary deployment consists of a separate deployment with your new version and a service that targets both your normal, stable deployment as well as your canary deployment.

```YAML
	apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: canary
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
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
```
- To create canry deployment

```bash
	kubectl create -f deployments/hello-canary.yaml
```

- Verify the canary deployment

```bash
	curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```


## Task 5: Blue-green deployments
Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.

Kubernetes achieves this by creating two separate deployments; one for the old "blue" version and one for the new "green" version. Use your existing hello deployment for the "blue" version. The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.


### The service

Use the existing hello service, but update it so that it has a selector app:hello, version: 1.0.0. The selector will match the existing "blue" deployment. But it will not match the "green" deployment because it will use a different version.

```yaml
	kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  selector:
    app: "hello"
    version: 1.0.0
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
      
```

- create a service 

```bash
	kubectl apply -f services/hello-blue.yaml
```

- Updating using Blue-Green deployment

```yaml
	apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
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
```

- Update service to point to new service

```bash
	kind: Service
apiVersion: v1
metadata:
  name: hello
spec:
  selector:
    app: hello
    version: 2.0.0
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      
      
	kubectl apply -f services/hello-green.yaml
```
