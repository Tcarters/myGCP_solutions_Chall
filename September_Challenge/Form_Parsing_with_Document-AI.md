


## Task 1. Enable the Document AI API


```bash
	gcloud services enable documentai.googleapis.com
	
	
	## install deps
	
	pip3 install --upgrade pandas
	
	
	pip3 install --upgrade google-cloud-documentai
	
	
```


## Task 2. Create a Form Parser processor

- Download an image doc and upload to the form parser


## Task 3. Download the sample form


```bash

	gcloud storage cp gs://cloud-samples-data/documentai/codelabs/form-parser/intake-form.pdf .
	
	## conform
	
	ls -ltr intake-form.pdf
	
	
```

## Task 4. Extract form key/value pairs

- Content of `form_parser.py`

```bash
	import pandas as pd
from google.cloud import documentai_v1 as documentai
def online_process(
    project_id: str,
    location: str,
    processor_id: str,
    file_path: str,
    mime_type: str,
) -> documentai.Document:
    """
    Processes a document using the Document AI Online Processing API.
    """
    opts = {"api_endpoint": f"{location}-documentai.googleapis.com"}
    # Instantiates a client
    documentai_client = documentai.DocumentProcessorServiceClient(client_options=opts)
    # The full resource name of the processor, e.g.:
    # projects/project-id/locations/location/processor/processor-id
    # You must create new processors in the Cloud Console first
    resource_name = documentai_client.processor_path(project_id, location, processor_id)
    # Read the file into memory
    with open(file_path, "rb") as image:
        image_content = image.read()
        # Load Binary Data into Document AI RawDocument Object
        raw_document = documentai.RawDocument(
            content=image_content, mime_type=mime_type
        )
        # Configure the process request
        request = documentai.ProcessRequest(
            name=resource_name, raw_document=raw_document
        )
        # Use the Document AI client to process the sample form
        result = documentai_client.process_document(request=request)
        return result.document
def trim_text(text: str):
    """
    Remove extra space characters from text (blank, newline, tab, etc.)
    """
    return text.strip().replace("\n", " ")
PROJECT_ID = "YOUR_PROJECT_ID"
LOCATION = "YOUR_PROJECT_LOCATION"  # Format is 'us' or 'eu'
PROCESSOR_ID = "FORM_PARSER_ID"  # Create processor in Cloud Console
# The local file in your current working directory
FILE_PATH = "form.pdf"
# Refer to https://cloud.google.com/document-ai/docs/processors-list
# for supported file types
MIME_TYPE = "application/pdf"
document = online_process(
    project_id=PROJECT_ID,
    location=LOCATION,
    processor_id=PROCESSOR_ID,
    file_path=FILE_PATH,
    mime_type=MIME_TYPE,
)
names = []
name_confidence = []
values = []
value_confidence = []
for page in document.pages:
    for field in page.form_fields:
        # Get the extracted field names
        names.append(trim_text(field.field_name.text_anchor.content))
        # Confidence - How "sure" the Model is that the text is correct
        name_confidence.append(field.field_name.confidence)
        values.append(trim_text(field.field_value.text_anchor.content))
        value_confidence.append(field.field_value.confidence)
# Create a Pandas Dataframe to print the values in tabular format.
df = pd.DataFrame(
    {
        "Field Name": names,
        "Field Name Confidence": name_confidence,
        "Field Value": values,
        "Field Value Confidence": value_confidence,
    }
)
print(df)
```


## Task 5. Parse tables

```bash

	## Download sample form with tables
	gcloud storage cp gs://cloud-samples-data/documentai/codelabs/form-parser/form_with_tables.pdf .
	
	ls -ltr form_with_tables.pdf
	
	#Extract Table Data

	touch table_parsing.py
	
```

- Content of file

```py
	# type: ignore[1]
"""
Uses Document AI online processing to call a form parser processor
Extracts the tables and data in the document.
"""
from os.path import splitext
from typing import List, Sequence
import pandas as pd
from google.cloud import documentai
def online_process(
    project_id: str,
    location: str,
    processor_id: str,
    file_path: str,
    mime_type: str,
) -> documentai.Document:
    """
    Processes a document using the Document AI Online Processing API.
    """
    opts = {"api_endpoint": f"{location}-documentai.googleapis.com"}
    # Instantiates a client
    documentai_client = documentai.DocumentProcessorServiceClient(client_options=opts)
    # The full resource name of the processor, e.g.:
    # projects/project-id/locations/location/processor/processor-id
    # You must create new processors in the Cloud Console first
    resource_name = documentai_client.processor_path(project_id, location, processor_id)
    # Read the file into memory
    with open(file_path, "rb") as image:
        image_content = image.read()
        # Load Binary Data into Document AI RawDocument Object
        raw_document = documentai.RawDocument(
            content=image_content, mime_type=mime_type
        )
        # Configure the process request
        request = documentai.ProcessRequest(
            name=resource_name, raw_document=raw_document
        )
        # Use the Document AI client to process the sample form
        result = documentai_client.process_document(request=request)
        return result.document
def get_table_data(
    rows: Sequence[documentai.Document.Page.Table.TableRow], text: str
) -> List[List[str]]:
    """
    Get Text data from table rows
    """
    all_values: List[List[str]] = []
    for row in rows:
        current_row_values: List[str] = []
        for cell in row.cells:
            current_row_values.append(
                text_anchor_to_text(cell.layout.text_anchor, text)
            )
        all_values.append(current_row_values)
    return all_values
def text_anchor_to_text(text_anchor: documentai.Document.TextAnchor, text: str) -> str:
    """
    Document AI identifies table data by their offsets in the entirety of the
    document's text. This function converts offsets to a string.
    """
    response = ""
    # If a text segment spans several lines, it will
    # be stored in different text segments.
    for segment in text_anchor.text_segments:
        start_index = int(segment.start_index)
        end_index = int(segment.end_index)
        response += text[start_index:end_index]
    return response.strip().replace("\n", " ")
PROJECT_ID = "qwiklabs-gcp-00-ad2a2c16c578"
LOCATION = "us"  # Format is 'us' or 'eu'
PROCESSOR_ID = "b708fdebcb3962fc"  # Create processor before running sample
# The local file in your current working directory
FILE_PATH = "form_with_tables.pdf"
# Refer to https://cloud.google.com/document-ai/docs/file-types
# for supported file types
MIME_TYPE = "application/pdf"
document = online_process(
    project_id=PROJECT_ID,
    location=LOCATION,
    processor_id=PROCESSOR_ID,
    file_path=FILE_PATH,
    mime_type=MIME_TYPE,
)
header_row_values: List[List[str]] = []
body_row_values: List[List[str]] = []
# Input Filename without extension
output_file_prefix = splitext(FILE_PATH)[0]
for page in document.pages:
    for index, table in enumerate(page.tables):
        header_row_values = get_table_data(table.header_rows, document.text)
        body_row_values = get_table_data(table.body_rows, document.text)
        # Create a Pandas Dataframe to print the values in tabular format.
        df = pd.DataFrame(
            data=body_row_values,
            columns=pd.MultiIndex.from_arrays(header_row_values),
        )
        print(f"Page {page.page_number} - Table {index}")
        print(df)
        # Save each table as a CSV file
        output_filename = f"{output_file_prefix}_pg{page.page_number}_tb{index}.csv"
        df.to_csv(output_filename, index=False)
        
```

-----------------------------------------------------------------------____END_OF_GAME_------------------------------------------


