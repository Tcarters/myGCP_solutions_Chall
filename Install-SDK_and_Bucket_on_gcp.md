## Setup Requirement

```bash

curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-440.0.0-linux-x86_64.tar.gz

gunzip google-cloud-cli-440.0.0-linux-x86_64.tar.gz 

tar -xf google-cloud-cli-440.0.0-linux-x86.tar.gz

./google-cloud-sdk/install.sh

./google-cloud-sdk/bin/gcloud init --console-only

./google-cloud-sdk/bin/gcloud config list all

```

### Task 1 : Create a bucket

- Set the region to `` us-west-1``
```bash
	gcloud config set compute/region us-west1
```

- We use the utility tool `` gsutil`` installed with the gcloud CLI tool associated to the make bucket *mb* command to create a bucket

```bash
	gsutil mb gs://mybucket1
```

### Task 2: Upload an object into our bucket

- Download first an image locally :
```bash
	curl https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg --output ada.jpg
```

- Upload the image to our bucket:
```bash
	gsutil cp ada.jpg gs://mybucket1
```

### Task 3: Download an object from bucket

```bash
	gsutil cp -r gs://YOUR-BUCKET-NAME/ada.jpg .
```

### Task 4: Copy an object to a folder in the bucket

- Use the gsutil cp command to create a folder called image-folder and copy the image (ada.jpg) into it...

```bash
	gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/
```

### Task 5. List contents of a bucket or folder

```bash
	gsutil ls gs://mybucket1
```

- List the details of a bucket

```bash
	gsutil ls -l gs://mybucket1/ada.jpg
```

### Task 6: Make ur object publicly accessible

```bash
	gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg
```

- Remove public access

```bash
	gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg
```

### Task 7: Delete object

```bash
	gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg
```
