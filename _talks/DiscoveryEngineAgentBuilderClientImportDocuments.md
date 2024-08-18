---
title: 'ConversationAI: Importing Documents'
collection: talks
type: "Using Discovery Engine Agent Builder Client to Import Documents into a Vertex AI Agent Datastore from Cloud Storage via Cloud Function"
permalink: /talks/DiscoveryEngineAgentBuilderClientImportDocuments
date: 2024-08-17
---

Objectives

1. Develop a Python code:  to use Discovery Engine Agent Builder Client Libraries to Import Documents into Agent Builder Data Stores from Cloud Storage
2. Deploy the Python code in Cloud Functions
3. Read Document from Cloud Storage into Vertex AI Agent Data Store
4. Test the DialogFlow CX Conversation Chatbot to be able to respond with the new data.

I am agoing to add data about enterprise arhictect. The DialogFlow CX is not able to provide a result since it does not have information from the data store on it.

![image](https://github.com/user-attachments/assets/7438a251-7c2c-4365-ad67-0ff7e85a67c3)

We will use automation to import the data and will then try again.

# 1. Python Client
These are the main library. I did not find much help from the library, so I had to read the documentation to figure this out.
```python
from google.api_core.client_options import ClientOptions
from google.cloud import discoveryengine_v1
from google.cloud.discoveryengine_v1 import ImportDocumentsRequest, GcsSource
```

## Some setup which we can read via runtime
```python
# If a parameter is not provided, use the default value.
project_id = request.args.get('project_id', "XXXX")
location = request.args.get('location', "us")
data_store_id = request.args.get('data_store_id', "all-githhub-as-text_XXXXX0")
gcs_uri = request.args.get('gcs_uri', "gs://XXXdatastore-dialogflowcx/enterprisearchitect.txt")
```

## API End Point
```python
# Determine the API endpoint based on the location.
    # If the location is not 'global', create a client_options object with the appropriate API endpoint.
    client_options = (
        ClientOptions(api_endpoint=f"{location}-discoveryengine.googleapis.com")
        if location != "global"
        else None
    )
 # Create a Discovery Engine Document Service client using the client_options.
    client = discoveryengine_v1.DocumentServiceClient(client_options=client_options)

    # Construct the full path to the branch within the specified data store.
    # This path is used as the 'parent' for the import request.
    parent = f"projects/{project_id}/locations/{location}/dataStores/{data_store_id}/branches/default_branch"
```

## The ImportDocumentsRequest
```python
 # Define the GCS source from which documents will be imported.
    # 'input_uris' is a list of GCS URIs pointing to the files you want to import.
    # 'data_schema' specifies the format of the documents, such as "content" or "csv".
    gcs_source = discoveryengine_v1.GcsSource(
        input_uris=[gcs_uri],
        data_schema="content",  # Replace with "csv" if the GCS file is in CSV format.
    )

    # Create an ImportDocumentsRequest object, specifying the parent path,
    # the GCS source, and the reconciliation mode.
    # The reconciliation mode "INCREMENTAL" means that only new or updated documents are imported.
    import_request = ImportDocumentsRequest(
        parent=parent,
        gcs_source=gcs_source,
        reconciliation_mode="INCREMENTAL"
    )

```
## Call the Discovery Engine API
```python
  # Call the Discovery Engine API to import documents based on the provided request.
  # 'operation.result()' waits for the import process to complete before proceeding.
  operation = client.import_documents(request=import_request)
  operation.result()  # Wait for the import to complete.
```

# 2. Cloud Funciton

there is a lack of time so I did not add any triggers into the cloud function. I have other demo for that. I will trigger the Cloud Funciton via Curl

![image](https://github.com/user-attachments/assets/ae91cfd2-7ab2-48ec-9f07-b3ccc8e01da1)

# 3. Read Document from Cloud Storage

![image](https://github.com/user-attachments/assets/6e7abc9e-a69d-4640-9e42-149584b13a28)'

The text is from Wikipedia on enterprise architect

![image](https://github.com/user-attachments/assets/9a2f36e9-aa5a-41bf-ba28-4c41d2e9f081)

## Running the Cloud Function
![image](https://github.com/user-attachments/assets/d2af86ec-db68-4697-8af3-4a22cf51ca84)

## Checking Data Store if the document was uploaded

Checking the activity tab
![image](https://github.com/user-attachments/assets/3c7792d9-7d2a-41b3-9ea9-bb3825a51d16)

checking the document

![image](https://github.com/user-attachments/assets/08182acd-73d2-4bee-97f0-b3488b666696)

# 4. Validating if the Dialog Flow has this updated and genterates a result.

now we have a answer

<img width="353" alt="image" src="https://github.com/user-attachments/assets/d18853d1-8602-4eb7-8a41-ee77227f3802">

I would like to compare the original just for kicks.

![image](https://github.com/user-attachments/assets/99cf5861-0a73-4c05-bf7e-e50d57ee41df)









