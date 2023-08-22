# Cloud Functions 2nd Gen: Qwik Start

## Task 1. Enable APIs


```bash
    export PROJECT_ID=$(gcloud config get-value project)

    export REGION=us-east4
    gcloud config set compute/region $REGION

    gcloud services enable \
  artifactregistry.googleapis.com \
  cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com \
  eventarc.googleapis.com \
  run.googleapis.com \
  logging.googleapis.com \
  pubsub.googleapis.com

```

## Task 2. Create an HTTP function

-Prepare files

```bash
    mkdir ~/hello-http && cd $_
    touch index.js && touch package.json


```

- Content of `hello-http/index.js`
```bash
    const functions = require('@google-cloud/functions-framework');
functions.http('helloWorld', (req, res) => {
  res.status(200).send('HTTP with Node.js in GCF 2nd gen!');
});


```

- Content of `hello-http/package.json`

```bash
    {
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^2.0.0"
  }
}
```

- Deploy the function

```bash
	gcloud functions deploy nodejs-http-function \
  --gen2 \
  --runtime nodejs16 \
  --entry-point helloWorld \
  --source . \
  --region $REGION \
  --trigger-http \
  --timeout 600s \
  --max-instances 1
```

- Test Function

```bash
	gcloud functions call nodejs-http-function \
  --gen2 --region $REGION
```


## Task 3. Create a Cloud Storage function

- Grant `pubsub.publisher` IAM role 

```bash
	PROJECT_NUMBER=$(gcloud projects list --filter="project_id:$PROJECT_ID" --format='value(project_number)')
SERVICE_ACCOUNT=$(gsutil kms serviceaccount -p $PROJECT_NUMBER)
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member serviceAccount:$SERVICE_ACCOUNT \
  --role roles/pubsub.publisher
```

- Create an App

```bash
	mkdir ~/hello-storage && cd $_
	touch index.js && touch package.json

	## Content of hello-storage/index.js
	const functions = require('@google-cloud/functions-framework');
functions.cloudEvent('helloStorage', (cloudevent) => {
  console.log('Cloud Storage event with Node.js in GCF 2nd gen!');
  console.log(cloudevent);
});

	## content of package.json
	{
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^2.0.0"
  }
}
```


- Deployment

```bash
	BUCKET="gs://gcf-gen2-storage-$PROJECT_ID"
gsutil mb -l $REGION $BUCKET

	
	## Deploy function 
	gcloud functions deploy nodejs-storage-function \
  --gen2 \
  --runtime nodejs16 \
  --entry-point helloStorage \
  --source . \
  --region $REGION \
  --trigger-bucket $BUCKET \
  --trigger-location $REGION \
  --max-instances 1
```


- Test

```bash
	echo "Hello World" > random.txt
	gsutil cp random.txt $BUCKET/random.txt
	
	gcloud functions logs read nodejs-storage-function \
  --region $REGION --gen2 --limit=100 --format "value(log)"
```


## Task 4. Create a Cloud Audit Logs function

- In this section, you will create a Node.js function that receives a Cloud Audit Log event when a Compute Engine VM instance is created. In response, it adds a label to the newly created VM, specifying the creator of the VM.

```bash
	## Setup
	1. From the Navigation Menu, go to IAM & Admin > Audit Logs.

	2. Find the Compute Engine API and click the check box next to it. If you are unable to find the API, search it on next page.
	
	3. n the info pane on the right, check Admin Read, Data Read, and Data Write log types and click Save.
	
	4. Grant the default Compute Engine service account the eventarc.eventReceiver IAM role:
	gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
  --role roles/eventarc.eventReceiver
```

- Get the code

```bash
	cd ~
	git clone https://github.com/GoogleCloudPlatform/eventarc-samples.git
	cd ~/eventarc-samples/gce-vm-labeler/gcf/nodejs
	
```

- Deployment

```bash
	gcloud functions deploy gce-vm-labeler \
  --gen2 \
  --runtime nodejs16 \
  --entry-point labelVmCreation \
  --source . \
  --region $REGION \
  --trigger-event-filters="type=google.cloud.audit.log.v1.written,serviceName=compute.googleapis.com,methodName=beta.compute.instances.insert" \
  --trigger-location $REGION \
  --max-instances 1
	
```

- Test 

```bash
	Create an VM
	gcloud compute instances describe instance-1 --zone us-east4-b
```


##  Task 5. Deploy different revisions

- Create 

```bash
	mkdir ~/hello-world-colored && cd $_touch main.py
	
	## Code
	import os
	color = os.environ.get('COLOR')
	def hello_world(request):
    		return f'<body style="background-color:{color}"><h1>Hello World!</h1></body>'

```

- Deployment 

```bash
	COLOR=orange
gcloud functions deploy hello-world-colored \
  --gen2 \
  --runtime python39 \
  --entry-point hello_world \
  --source . \
  --region $REGION \
  --trigger-http \
  --allow-unauthenticated \
  --update-env-vars COLOR=$COLOR \
  --max-instances 1
  
```


## Task 6. Set up minimum instances

- Create

```bash
	mkdir ~/min-instances && cd $_
	touch main.go
	
```

- Deployment

```bash
	gcloud functions deploy slow-function \
  --gen2 \
  --runtime go116 \
  --entry-point HelloWorld \
  --source . \
  --region $REGION \
  --trigger-http \
  --allow-unauthenticated \
  --max-instances 4
  
  	## Test function
  	
  	gcloud functions call slow-function \
  --gen2 --region $REGION
  
```

## Task 7. Create a function with concurrency

- Test without concurrency

```bash
	SLOW_URL=$(gcloud functions describe slow-function --region $REGION --gen2 --format="value(serviceConfig.uri)")
	
	##Send 10 concurrent requests to the slow function 
	## Git of hey cmd: https://github.com/rakyll/hey
	
	hey -n 10 -c 10 $SLOW_URL
	
```

- Deployment

```bash
	gcloud functions deploy slow-concurrent-function \
  --gen2 \
  --runtime go116 \
  --entry-point HelloWorld \
  --source . \
  --region $REGION \
  --trigger-http \
  --allow-unauthenticated \
  --min-instances 1 \
  --max-instances 4 
```

- Test with concurrency

```bash
	SLOW_CONCURRENT_URL=$(gcloud functions describe slow-concurrent-function --region $REGION --gen2 --format="value(serviceConfig.uri)")
	
  hey -n 10 -c 10 $SLOW_CONCURRENT_URL

```

------------------------------------------------------🎉END-GAME🎉-------------------------------------------------------------------------------------------------------------------------
