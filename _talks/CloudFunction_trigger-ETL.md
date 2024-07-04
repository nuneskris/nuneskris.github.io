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
      			 MyCallBigQueryToLoadLocalFile.loadLocalFile(
      					 cloudStorageBody.getDatasetName(), 
      					 cloudStorageBody.getTableViewName() , cloudStorageBody.getFileName()); 
      	    }
      	  }
      	}





