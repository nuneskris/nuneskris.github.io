---
title: "Trigger a function when a new file is uploaded to cloud storage"
collection: talks
type: "Do not wait for a schedule to trigger a pipeline"
permalink: /talks/CloudFunction_trigger-ETL
date: 2024-03-01
---

Even in 2024, I am seeing pipelines waiting for a schedule rather than being automatically triggered when the data file arrives.

I am called to review troubled data engineering pipelines and I have encountered pipelines in trouble because transformation was waiting on a schedule and losing precious time. Twice I implemented a simple change (once on AWS and once on GCP) and that was enough to solve 50% of its problems or atleast buy enough time before others could be resolved.

<img width="636" alt="image" src="https://github.com/nuneskris/nuneskris.github.io/assets/82786764/63862e3c-d963-44b8-bd31-b07e07a0a842">

## Component: Cloud Function

### Main Function Class
```python
	package com.java.kfn.study.gcp.cloudfunction.invoke;
	import java.io.IOException;
	import java.nio.charset.StandardCharsets;
	import java.util.logging.Logger;
	import org.apache.http.client.ClientProtocolException;
	import com.google.cloud.functions.CloudEventsFunction;
	import io.cloudevents.CloudEvent;
	// Step 1: Implement the com.google.cloud.functions.CloudEventsFunction. We will define how this is called during deployment
	public class AutomaticFileEventHandling implements CloudEventsFunction {
	private static final Logger logger = Logger.getLogger(AutomaticFileEventHandling.class.getName());

	  @Override
	  public void accept(CloudEvent event) throws ClientProtocolException, IOException, InterruptedException {	 
		 if (event.getData() == null) {
		      logger.warning("No data found in cloud event payload!");
		      return;
		 } else  {
			 // this is the  string data of the cloud event which triggered the cloud function.
			 // In our case this is the event which was triggered when the file arrived at the cloud storage
			 // Step2:  Read the event
			 String cloudEventData = new String(event.getData().toBytes(), StandardCharsets.UTF_8);
			 logger.info("Event: " + event.getId() +", Event Type: " + event.getType());
			 //Step 4: We would need to parse the event string. We can get information like bucket name, file name, event type, bucket meta data etc
			 MyCloudStorageEventHandler cloudStorageBody = new MyCloudStorageEventHandler(cloudEventData);
			 //Step 5: Simple Java Class to Load data into Big Query
			 MyCallBigQueryToLoadLocalFile bqload = new MyCallBigQueryToLoadLocalFile();
			 bqload.loadLocalFile(
					 cloudStorageBody.getDatasetName(), 
					 cloudStorageBody.getTableViewName() , 
					 cloudStorageBody.getFilePath());
	     
	    }
	  
	  }
}
```

### Component: Read the event.
We are able to read the file, bucket and also any meta data associated with the file which was ingested. 
To make this interesting we will be adding the BQ dataset and table name within the metadata of the file.

	package com.java.kfn.study.gcp.cloudfunction.invoke;
	import java.util.Map;
	import com.google.events.cloud.storage.v1.StorageObjectData;
	import com.google.protobuf.InvalidProtocolBufferException;
	import com.google.protobuf.util.JsonFormat;
	
	public class MyCloudStorageEventHandler {

		private String tableViewName;
		private Object bucket;
		private String fileName;
		private String datasetName;
		
		public String getFilePath() {
			return "gs://"+this.bucket+"/"+this.fileName;
		}
	
		public MyCloudStorageEventHandler(String cloudEventData) throws InvalidProtocolBufferException {
			 // StorageObjectData is a helper class provided by GCP to parse an GCP Storage Event string.
			 StorageObjectData.Builder builder = StorageObjectData.newBuilder(); 
			 JsonFormat.parser().merge(cloudEventData, builder);
			 StorageObjectData storageObjectData = builder.build();
			 // Storage Object (Bucket and file name 
			 this.bucket=storageObjectData.getBucket();
			 this.fileName=storageObjectData.getName();  
			 // We can even parse metadata of the storage object. 
			 // To make it interesting, we will be parsing the target BQ Dataset and table name from the storage metadata
			 Map<String, String> metadataMap = storageObjectData.getMetadataMap();
			 this.tableViewName = metadataMap.get("table_view_name");
			 this.datasetName = metadataMap.get("dataset_name");
			 
	   }
		public String getTableViewName() {
			return this.tableViewName;
		}
		public Object getBucket() {
			return this.bucket;
		}
	
		public String getFileName() {
			return this.fileName;
		}
	
		public String getDatasetName() {
			return this.datasetName;
		}
	}


### Component: Load data in Big Query


	package com.java.kfn.study.gcp.cloudfunction.invoke;
	import java.io.IOException;
	import com.google.cloud.bigquery.BigQuery;
	import com.google.cloud.bigquery.BigQueryException;
	import com.google.cloud.bigquery.BigQueryOptions;
	import com.google.cloud.bigquery.FormatOptions;
	import com.google.cloud.bigquery.Job;
	import com.google.cloud.bigquery.JobInfo;
	import com.google.cloud.bigquery.LoadJobConfiguration;
	import com.google.cloud.bigquery.TableId;
	
	public class MyCallBigQueryToLoadLocalFile {
	
		 public  void loadLocalFile(
			      String datasetName, String tableName, String csv_Path)
			      throws IOException, InterruptedException {
			    try {
			      // Initialize client that will be used to send requests. This client only needs to be created
			      // once, and can be reused for multiple requests.
			      BigQuery bigquery = BigQueryOptions.getDefaultInstance().getService();
	
			      TableId tableId = TableId.of(datasetName, tableName);
			      LoadJobConfiguration loadConfig = LoadJobConfiguration.newBuilder(tableId, csv_Path).setFormatOptions(FormatOptions.csv()).build();
			      Job loadJob = bigquery.create(JobInfo.of(loadConfig));
			      loadJob = loadJob.waitFor();
			      if (loadJob.isDone()) {
			    	    System.out.println("Data successfully loaded into BigQuery table");
			    	} else {
			    	    System.out.println("Failed to load data into BigQuery table: " + loadJob.getStatus().getError());
			    	}
	
			       } catch (BigQueryException e) {
			      System.out.println("Local file not loaded. \n" + e);
			      throw e;
			    }
			  }
	
	}


## Cloud Function deployment

	##### some setup commands
	gcloud init
	export ORAC_GCP_ETLPIPELINE_BUCKET_NAME=injest_study
	export ORAC_GCP_ETLPIPELINE_BUCKET_FOLDER=gs://${ORAC_GCP_ETLPIPELINE_BUCKET_NAME}
	
	##### delete cloud function if it already exists.
	gcloud functions delete gcp-ingest-finalize-function-ETL --gen2 --region us-west1 
	
	##### this indicates to deploy the cloud function within class, com.java.kfn.study.gcp.cloudfunction.invoke.AutomaticFileEventHandling
	##### trigger event: google.cloud.storage.object.v1.finalized
	##### bucket ORAC_GCP_ETLPIPELINE_BUCKET_NAME
	##### gen2
	
	echo ${ORAC_GCP_ETLPIPELINE_BUCKET_FOLDER}
	gcloud functions deploy gcp-ingest-finalize-function-ETL \
	--gen2 \
	--runtime=java17 \
	--region=us-west1 \
	--source=. \
	--entry-point=com.java.kfn.study.gcp.cloudfunction.invoke.AutomaticFileEventHandling \
	--memory=512MB \
	--trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
	--trigger-event-filters="bucket=${ORAC_GCP_ETLPIPELINE_BUCKET_NAME}"

## Running the demo

We would need to ensure we have a bucket and a BigQuery Dataset and Table. Preload the schema to make thigns simple.

	##### ingest file into cloud storage
	gsutil -h x-goog-meta-table_view_name:allrounders \
		   -h x-goog-meta-dataset_name:cricketdb \
		   -h x-goog-meta-extract_last_modified_date:2024-03-07T14:47:24.899+00:00 \
		    cp curatedAllrounders.csv ${ORAC_GCP_ETLPIPELINE_BUCKET_FOLDER}/curatedAllroundersXXXX.csv
		    
	##### Read the logs
	gcloud functions logs read gcp-ingest-finalize-function-ETL --region us-west1 --gen2 --limit=5

## Results.
#### Deployed Cloud Function.
The details indicate the trigger and the execution function which we developed and deployed earlier

![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/d5fb22b3-2202-469b-b5b4-25ffa31a0892)

#### File ingested into Cloud Storage
On arrival, the trigger event invoked the cloud function.
![image](https://github.com/nuneskris/nuneskris.github.io/assets/82786764/09fc5454-4b16-4d45-b155-82217b16d64b)

#### Data Loaded in Big Query Table
 <img width="1233" alt="image" src="https://github.com/nuneskris/nuneskris.github.io/assets/82786764/daf1037d-7778-4cad-95e4-45f9aad6b4b4">

 

