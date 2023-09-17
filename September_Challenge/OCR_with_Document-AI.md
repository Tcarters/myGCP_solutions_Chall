# Optical Character Recognition (OCR) with Document AI (Python)


## Task 1. Enable the Document AI API

```bash
	gcloud services enable documentai.googleapis.com
	gcloud services enable storage.googleapis.com

```

## Task 2. Create and test a processor

- Upload a pdf to Document AI
```bash
	
```


## Task 3. Authenticate API requests

```bash

	export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)
	
	gcloud iam service-accounts create my-docai-sa \
  --display-name "my-docai-service-account"
  
  	## Give service account permissions to acces Document AI:
  	
  	gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
    --member="serviceAccount:my-docai-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role="roles/documentai.admin"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
    --member="serviceAccount:my-docai-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role="roles/storage.admin"
gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
    --member="serviceAccount:my-docai-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
    --role="roles/serviceusage.serviceUsageConsumer"
    
    ## Create credentials 
	gcloud iam service-accounts keys create ~/key.json \
  --iam-account  my-docai-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com


	## Set env varibales:
	export GOOGLE_APPLICATION_CREDENTIALS=$(realpath key.json)
	  
```


## Task 4. Install the client library


```bash
	pip3 install --upgrade google-cloud-documentai
	pip3 install --upgrade google-cloud-storage
	
	## Upload the sample PDF to Cloud Shell

	gcloud storage cp gs://cloud-samples-data/documentai/codelabs/ocr/Winnie_the_Pooh_3_Pages.pdf .

```
	
## Task 5. Make an online processing request

```bash
	touch online_processing.py
	
	
	## Content of file:
	
	from google.api_core.client_options import ClientOptions
from google.cloud import documentai_v1 as documentai
PROJECT_ID = "YOUR_PROJECT_ID"
LOCATION = "YOUR_PROJECT_LOCATION"  # Format is 'us' or 'eu'
PROCESSOR_ID = "YOUR_PROCESSOR_ID"  # Create processor in Cloud Console
# The local file in your current working directory
FILE_PATH = "Winnie_the_Pooh_3_Pages.pdf"
# Refer to https://cloud.google.com/document-ai/docs/file-types
# for supported file types
MIME_TYPE = "application/pdf"
# Instantiates a client
docai_client = documentai.DocumentProcessorServiceClient(
    client_options=ClientOptions(api_endpoint=f"{LOCATION}-documentai.googleapis.com")
)
# The full resource name of the processor, e.g.:
# projects/project-id/locations/location/processor/processor-id
# You must create new processors in the Cloud Console first
RESOURCE_NAME = docai_client.processor_path(PROJECT_ID, LOCATION, PROCESSOR_ID)
# Read the file into memory
with open(FILE_PATH, "rb") as image:
    image_content = image.read()
# Load Binary Data into Document AI RawDocument Object
raw_document = documentai.RawDocument(content=image_content, mime_type=MIME_TYPE)
# Configure the process request
request = documentai.ProcessRequest(name=RESOURCE_NAME, raw_document=raw_document)
# Use the Document AI client to process the sample form
result = docai_client.process_document(request=request)
document_object = result.document
print("Document processing complete.")
print(f"Text: {document_object.text}")


```

- 


## Task 6. Make a batch processing request

- Upload PDF to Cloud Storage


```bash

	gcloud storage buckets create gs://$GOOGLE_CLOUD_PROJECT
gcloud storage cp gs://cloud-samples-data/documentai/codelabs/ocr/Winnie_the_Pooh.pdf gs://$GOOGLE_CLOUD_PROJECT/


	## Create batch_processing.py
	
	import re
from typing import List
from google.api_core.client_options import ClientOptions
from google.cloud import documentai_v1 as documentai
from google.cloud import storage
PROJECT_ID = "YOUR_PROJECT_ID"
LOCATION = "YOUR_PROJECT_LOCATION"  # Format is 'us' or 'eu'
PROCESSOR_ID = "YOUR_PROCESSOR_ID"  # Create processor in Cloud Console
# Format 'gs://input_bucket/directory/file.pdf'
GCS_INPUT_URI = "gs://cloud-samples-data/documentai/codelabs/ocr/Winnie_the_Pooh.pdf"
INPUT_MIME_TYPE = "application/pdf"
# Format 'gs://output_bucket/directory'
GCS_OUTPUT_URI = "YOUR_OUTPUT_BUCKET_URI"
# Instantiates a client
docai_client = documentai.DocumentProcessorServiceClient(
    client_options=ClientOptions(api_endpoint=f"{LOCATION}-documentai.googleapis.com")
)
# The full resource name of the processor, e.g.:
# projects/project-id/locations/location/processor/processor-id
# You must create new processors in the Cloud Console first
RESOURCE_NAME = docai_client.processor_path(PROJECT_ID, LOCATION, PROCESSOR_ID)
# Cloud Storage URI for the Input Document
input_document = documentai.GcsDocument(
    gcs_uri=GCS_INPUT_URI, mime_type=INPUT_MIME_TYPE
)
# Load GCS Input URI into a List of document files
input_config = documentai.BatchDocumentsInputConfig(
    gcs_documents=documentai.GcsDocuments(documents=[input_document])
)
# Cloud Storage URI for Output directory
gcs_output_config = documentai.DocumentOutputConfig.GcsOutputConfig(
    gcs_uri=GCS_OUTPUT_URI
)
# Load GCS Output URI into OutputConfig object
output_config = documentai.DocumentOutputConfig(gcs_output_config=gcs_output_config)
# Configure Process Request
request = documentai.BatchProcessRequest(
    name=RESOURCE_NAME,
    input_documents=input_config,
    document_output_config=output_config,
)
# Batch Process returns a Long Running Operation (LRO)
operation = docai_client.batch_process_documents(request)
# Continually polls the operation until it is complete.
# This could take some time for larger files
# Format: projects/PROJECT_NUMBER/locations/LOCATION/operations/OPERATION_ID
print(f"Waiting for operation {operation.operation.name} to complete...")
operation.result()
# NOTE: Can also use callbacks for asynchronous processing
#
# def my_callback(future):
#   result = future.result()
#
# operation.add_done_callback(my_callback)
print("Document processing complete.")
# Once the operation is complete,
# get output document information from operation metadata
metadata = documentai.BatchProcessMetadata(operation.metadata)
if metadata.state != documentai.BatchProcessMetadata.State.SUCCEEDED:
    raise ValueError(f"Batch Process Failed: {metadata.state_message}")
documents: List[documentai.Document] = []
# Storage Client to retrieve the output files from GCS
storage_client = storage.Client()
# One process per Input Document
for process in metadata.individual_process_statuses:
    # output_gcs_destination format: gs://BUCKET/PREFIX/OPERATION_NUMBER/0
    # The GCS API requires the bucket name and URI prefix separately
    output_bucket, output_prefix = re.match(
        r"gs://(.*?)/(.*)", process.output_gcs_destination
    ).groups()
    # Get List of Document Objects from the Output Bucket
    output_blobs = storage_client.list_blobs(output_bucket, prefix=output_prefix)
    # DocAI may output multiple JSON files per source file
    for blob in output_blobs:
        # Document AI should only output JSON files to GCS
        if ".json" not in blob.name:
            print(f"Skipping non-supported file type {blob.name}")
            continue
        print(f"Fetching {blob.name}")
        # Download JSON File and Convert to Document Object
        document = documentai.Document.from_json(
            blob.download_as_bytes(), ignore_unknown_fields=True
        )
        documents.append(document)
# Print Text from all documents
# Truncated at 100 characters for brevity
for document in documents:
    print(document.text[:100])
    
    
```


## Task 7. Make a batch processing request for a directory

- Create file `batch_process_documents.py`

```bash
	import re
from typing import List
from google.api_core.client_options import ClientOptions
from google.cloud import documentai_v1 as documentai
from google.cloud import storage
PROJECT_ID = "qwiklabs-gcp-02-6056304a7edd"
LOCATION = "us"  # Format is 'us' or 'eu'
PROCESSOR_ID = "f3fe54e872c4721b"  # Create processor in Cloud Console
# Format 'gs://input_bucket/directory'
GCS_INPUT_PREFIX = "gs://qwiklabs-gcp-02-6056304a7edd/multi-document"
# Format 'gs://output_bucket/directory'
GCS_OUTPUT_URI = "gs://qwiklabs-gcp-02-6056304a7edd"
# Instantiates a client
docai_client = documentai.DocumentProcessorServiceClient(
    client_options=ClientOptions(api_endpoint=f"{LOCATION}-documentai.googleapis.com")
)
# The full resource name of the processor, e.g.:
# projects/project-id/locations/location/processor/processor-id
# You must create new processors in the Cloud Console first
RESOURCE_NAME = docai_client.processor_path(PROJECT_ID, LOCATION, PROCESSOR_ID)
# Cloud Storage URI for the Input Directory
gcs_prefix = documentai.GcsPrefix(gcs_uri_prefix=GCS_INPUT_PREFIX)
# Load GCS Input URI into Batch Input Config
input_config = documentai.BatchDocumentsInputConfig(gcs_prefix=gcs_prefix)
# Cloud Storage URI for Output directory
gcs_output_config = documentai.DocumentOutputConfig.GcsOutputConfig(
    gcs_uri=GCS_OUTPUT_URI
)
# Load GCS Output URI into OutputConfig object
output_config = documentai.DocumentOutputConfig(gcs_output_config=gcs_output_config)
# Configure Process Request
request = documentai.BatchProcessRequest(
    name=RESOURCE_NAME,
    input_documents=input_config,
    document_output_config=output_config,
)
# Batch Process returns a Long Running Operation (LRO)
operation = docai_client.batch_process_documents(request)
# Continually polls the operation until it is complete.
# This could take some time for larger files
# Format: projects/PROJECT_NUMBER/locations/LOCATION/operations/OPERATION_ID
print(f"Waiting for operation {operation.operation.name} to complete...")
operation.result()
# NOTE: Can also use callbacks for asynchronous processing
#
# def my_callback(future):
#   result = future.result()
#
# operation.add_done_callback(my_callback)
print("Document processing complete.")
# Once the operation is complete,
# get output document information from operation metadata
metadata = documentai.BatchProcessMetadata(operation.metadata)
if metadata.state != documentai.BatchProcessMetadata.State.SUCCEEDED:
    raise ValueError(f"Batch Process Failed: {metadata.state_message}")
documents: List[documentai.Document] = []
# Storage Client to retrieve the output files from GCS
storage_client = storage.Client()
# One process per Input Document
for process in metadata.individual_process_statuses:
    # output_gcs_destination format: gs://BUCKET/PREFIX/OPERATION_NUMBER/0
    # The GCS API requires the bucket name and URI prefix separately
    output_bucket, output_prefix = re.match(
        r"gs://(.*?)/(.*)", process.output_gcs_destination
    ).groups()
    # Get List of Document Objects from the Output Bucket
    output_blobs = storage_client.list_blobs(output_bucket, prefix=output_prefix)
    # DocAI may output multiple JSON files per source file
    for blob in output_blobs:
        # Document AI should only output JSON files to GCS
        if ".json" not in blob.name:
            print(f"Skipping non-supported file type {blob.name}")
            continue
        print(f"Fetching {blob.name}")
        # Download JSON File and Convert to Document Object
        document = documentai.Document.from_json(
            blob.download_as_bytes(), ignore_unknown_fields=True
        )
        documents.append(document)
# Print Text from all documents
# Truncated at 100 characters for brevity
for document in documents:
    print(document.text[:100])
    
```

--------------------------------------------------------------------END_GAME_-----------------------------------------------------------------------------------
