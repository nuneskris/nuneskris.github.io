---
title: "Extract: Batch Transfer From Onprem To Cloud (GCP)"
collection: talks
permalink: /talks/Extract-Batch-Transfer
date: 2024-06-29
---

<img width="354" alt="image" src="/images/talks/BatchTransferCron.png">


Typicall workloads begin with flatfiles which are extracted from the source system which are either in CSV or Parquet. We will require to transfer these files into the cloud. The solution is very straight forward
* Python gcloud sdk to upload the files
* Cron job to schedule the job

However, many would say that this solution is not enterprise grade. I will use the architecture pillars and enhance the subsytems to deliver a system which is secure, reliable, performant and cost optimized while conforming to operational excelence standards.

#  Automation and Scheduling

We need a mechanism which can be easily be scheduled. We are basically trying to not rely on complex orchestration tools which either need to be deployed on-prem or struggle with a solution which needs to get across the cloud network into the on-prem which can be a nightmare with threat controls.

Cron is a tried and tested option to schedule jobs. With proper documentation and maintenance, it can be a very reliable solution.

![image](https://github.com/user-attachments/assets/3b1349f8-1a24-49c4-9738-b3e3eccad443)

#### First we would need to find the correct Python path
```console
which python3
```

#### Edit the crontab
We would need to edit and add the schedule to the crontab
```console
crontab -e
```  

#### Update the cron job entry with the correct Python path
```console
*/5 * * * * /<path to python>/python3 /<path to code>/upload_to_gcs_scheduler.py >> /<path to log>/upload_to_gcs_scheduler.log 2>&1
```

#### We can verify the cron job
```console
    crontab -l
```

<img width="1020" alt="image" src="https://github.com/user-attachments/assets/5d17d42e-67ce-414c-9eed-e5723edb8831">

## logging
An often forgotten aspect of substems which support the pipeline is the logging. Since we have complete control on the susbystem, we can customize logging based on our needs. Python supports simple solution for loggig.

```python
import logging
.
.
# Setup logging
logging.basicConfig(filename='/Users/xxxxx/DataEngineering/ETL/Collect/Ingest/GCPUtilCopy/logfile.log',
                    level=logging.INFO,
                    format='%(asctime)s %(levelname)s %(message)s')
.
.
# logging
logging.info(f"INFO: {datetime.now().strftime('%Y%m%d%H%M%S')}: File {local_file_path} uploaded to {destination_blob_name}")
```


•Alerting: 

### Alerting: Pub/Sub Notification

Output Logs: Redirect cron job output to log files and monitor these logs for any errors or anomalies.
Alerting: Set up email alerts or integrate with monitoring tools (e.g., Prometheus, Grafana) to notify when the job fails.

### Automate Error Handling (Reliability)
Errors can happen because of network issues. There 2 points which we need to take into considertion. 
1. Transfer of Data
2. Sending Notificaiton on the the job.

#### Automate Error Handling (Reliability): Retry Logic
Implementing retry logic within the Python script to handle transient errors is straight forward. This can be done using libraries like tenacity for retries.

****Retry logic for sending Notificaiton on the the job****

```python
# retry after 5 mins, 5 times.
@tenacity.retry(wait=tenacity.wait_fixed(60), stop=tenacity.stop_after_attempt(5))
def publish_message(project_id, topic_id, message):
```
Testing the retry logic by turning of the internet. There is a pause for 5 mins and retries. It was able to successfully send the message on the retry after I turned on the internet.
<img width="612" alt="image" src="https://github.com/user-attachments/assets/5d0c41e6-7d73-423f-9a66-41ada488cecd">

****Retry logic for Transfer of Data****

```python
@tenacity.retry(wait=tenacity.wait_fixed(5), stop=tenacity.stop_after_attempt(5))
def upload_to_gcs(local_file_path, bucket_name, destination_blob_name, json_credentials_path):
```

#### Automate Error Handling (Reliability): Error Logging
Log errors and successes for monitoring and debugging purposes. Based on compliance needs to be would to build robust solutions to ensure logs are stored persistently and can be accessed.

We have 2 solutions: 

****1. Local On-Prem server soltution****

We can locally log data locally in a file for maintenance short term maintainance.

Step 1: Add print statements 
```python

```



# Design


# Run 

### Parquet Files Loaded in the Onprem extract source location.

![image](https://github.com/user-attachments/assets/b0105be3-9ba7-4cf8-8005-95da0838fe8c)
