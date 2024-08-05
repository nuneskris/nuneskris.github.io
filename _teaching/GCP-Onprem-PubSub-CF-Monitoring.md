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
1. Create a Cloud Function to be triggered when the a message is received by Pub/Sub
2. Cloud Function to log the message using Cloud Logging
3. Use Cloud Monitoring to create a Metric and Policy to monitor for a message arrival.
4. An Cloud Alert would need to send a email notificaiton when a message does not arrive within a timeframe.

# 1. Create a Cloud Function

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

## Deploy the Cloud Function
```console
gcloud functions deploy python-onprem-logreader-pubsub-function \
--gen2 \
--runtime=python312 \
--region=us-west1 \
--source=. \
--entry-point=subscribe \
--trigger-topic=topic-play-kfnstudy-trigger-cloudFunction
```
