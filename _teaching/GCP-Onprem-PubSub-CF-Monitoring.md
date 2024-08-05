---
title: "Building Reliability in On-prem Data Upload Jobs Through Log Monitoring"
collection: teaching
type: "Monitoring"
permalink: /teaching/GCP-Onprem-PubSub-CF-Monitoring
venue: "GCP"
date: 2024-06-03
---

<img width="612" alt="image" src="https://github.com/user-attachments/assets/72c6f1fa-7ec7-4fac-95c4-ad12c1892c59">

Pub/Sub does not natively log message data access events. Instead, it logs administrative actions and performance metrics. 
For monitoring the actual message flow (such as ensuring that messages are being published and processed), you'll need to implement a logging mechanism that writes to Cloud Logging whenever a message is published or processed.
We will use Cloud Funcion which is triggered by the Pub/Sub message is received and it will be used to log messages. We can expand this Cloud Function to customize messaging for multiple scenarios.

> We typically know how to handle job failures messages by using exception handling. However, we can expect on-prem uplaod job success or failure only if the job was initited. 

We would need to be able to register whether or not a job was initiated. 
In this scenario we would want to know that the on-prem job was initiated successfully. If we do not receive a job initaited message, it means that the Job scheduler failed and we would need to fix the problem.

# Objectives
1. Cloud Function to log the message using Cloud Logging
2. Deploy Cloud Function to be triggered by Pub/Sub when the a message is received 
3. Use Cloud Monitoring to create a Metric and Policy to monitor for a message arrival.
4. A Cloud Alert would need to send a email notificaiton when a message does not arrive within a timeframe.
5. Workflow Process to create an incident and close the incident when the condition is resolved. 

# 1. Cloud Function to log the message using Cloud Logging

```python
import base64
import google.cloud.logging
from cloudevents.http import CloudEvent
import functions_framework

# Triggered from a message on a Cloud Pub/Sub topic.
@functions_framework.cloud_event
def subscribe(cloud_event: CloudEvent) -> None:
    # Create a Cloud Logging Client
    client = google.cloud.logging.Client()
    client.setup_logging()
    # Create a Logger
    logger = client.logger("cronjob_erp_salesorderitems_pubsub_logger")
    # Create a Log Message
    message = base64.b64decode(cloud_event.data["message"]["data"]).decode()
    # Log the message
    logger.log_text(message)
```

This Cloud Function requires 2 libraries which we would need to configure in a ***requirements.txt***
```console
google-cloud-logging
pytest==8.2.0
```

# 2. Deploy Cloud Function to be triggered by Pub/Sub
```console
gcloud functions deploy python-onprem-logreader-pubsub-function \
--gen2 \
--runtime=python312 \
--region=us-west1 \
--source=. \
--entry-point=subscribe \
--trigger-topic=topic-play-kfnstudy-trigger-cloudFunction
```

## Running a test on Cloud Function by publishing a message to pub-sub

We are running a client from the [demo](https://nuneskris.github.io/teaching/GCloudSDK-Storage-PubSub) where we used a cron job to schedule a gcloud python client to copy files to Google storage and sent a notification to Pub/Sub that the client was triggered.

Message published from client:

```console
    INFO: 20240804165246:durable-pipe-431319-g7:Scheduled Job Initated; Ingest ERP Data - SalesOrderItems;20240804165246 Published message ID: 11920180892590542
```
Pub/Sub Subscription

![image](https://github.com/user-attachments/assets/82a2d563-2f09-4b11-a8d5-5bd087486019)
           
# 3. Cloud Monitoring to create a Metric and Policy
We can confirm that the message was logged by Cloud Logging by Google Cloud Monitoring

![image](https://github.com/user-attachments/assets/61837e38-1d93-441f-9ee9-8382fd195d0e)

## Create a metric

```json
resource.type="cloud_function"
resource.labels.function_name="python-onprem-logreader-pubsub-function"
resource.labels.project_id="durable-pipe-431319-g7"
resource.labels.region="us-west1"
logName="projects/durable-pipe-431319-g7/logs/cronjob_erp_salesorderitems_pubsub_logger"
textPayload:"Scheduled Job Initated; Ingest ERP Data - SalesOrderItems"
```

![image](https://github.com/user-attachments/assets/c0b01e52-7282-42a4-b42c-f5d89ed33d06)

![image](https://github.com/user-attachments/assets/5e6a9849-ae17-4427-a607-2361d9a0f446)

## Creating a policy on the metric

![image](https://github.com/user-attachments/assets/f29699f1-090e-40cf-bf9f-8aa5d04470fb)

We also configure a alert to be triggered based on a metric absence. If there is no alert we will trigger a notification

![image](https://github.com/user-attachments/assets/fe97c255-52e7-4249-91b5-682a645a8609)

I defined a notification channnel based on sending an email.

![image](https://github.com/user-attachments/assets/30309718-725b-4012-b764-0d92e024b2c7)


# 4. Alert and Notification

There was no alert in 5 mins and an alert was crated

![image](https://github.com/user-attachments/assets/4d45902d-7a76-4dda-af4f-0dc03a7548be)

An email was also triggered

![image](https://github.com/user-attachments/assets/b57da63a-20c9-4ad9-bccb-039f640784aa)

# 5. Workflow Process to create an incident and close the incident

![image](https://github.com/user-attachments/assets/48431e87-2e92-4e3d-8eed-329074fe2f5a)

### The incident can be acknowledged.

![image](https://github.com/user-attachments/assets/e1fa3b42-e23a-479c-9274-1fd3abd8ce9c)

### The incident only be resolved when the condition is met. A pubsub message is required to close the message.

![image](https://github.com/user-attachments/assets/86dbbe4a-3a15-48ad-b9a4-7139c72d9295)

### Publishing a message and closing the incindent.

![image](https://github.com/user-attachments/assets/c0575e09-58cb-47ae-b438-2afc296a5f45)

![image](https://github.com/user-attachments/assets/2c667c05-ecd8-4ec9-99f7-c271826454f8)

We are able to close the incident once we receive a message.
