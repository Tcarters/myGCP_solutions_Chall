
### Create a Function

- Set default region

```bash
	gcloud config set compute/region us-central1
	mkdir gcf_hello_world 
	cd gcf_hello_world
	
	cat <<EOF>>
	
	/**
* Background Cloud Function to be triggered by Pub/Sub.
* This function is exported by index.js, and executed when
* the trigger topic receives a message.
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";
console.log(`My Cloud Function: ${name}`);
};

	
	EOF
```



### Task 2. Create a cloud storage bucket

- Create a new cloud storage bucket:

```bash
	gsutil mb -p [PROJECT_ID] gs://[BUCKET_NAME]
```


### Task 3. Deploy your function
-When deploying a new function, you must specify --trigger-topic, --trigger-bucket, or --trigger-http. When deploying an update to an existing function, the function keeps the existing trigger unless otherwise specified.

For this lab, you'll set the --trigger-topic as hello_world.


- Deploy a function to a pub/sub topic named `` hello_world``

```bash
	gcloud functions deploy helloWorld \
  --stage-bucket [BUCKET_NAME] \
  --trigger-topic hello_world \
  --runtime nodejs20
```

- Verify Status of Fuction

```bash
	gcloud functions describe helloWorld
```


### Task 4. Test the function

- Test that the Function writes a message to cloud log:

```bash
	DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
```

### Task 5: View Logs

```bash
	gcloud functions logs read helloWorld
```

- Serverless lets you write and deploy code without the hassle of managing the underlying infrastructure.
