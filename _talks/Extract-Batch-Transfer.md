---
title: "Extract: Batch Transfer From Onprem To Cloud (GCP)"
collection: talks
permalink: /talks/Extract-Batch-Transfer
date: 2024-06-29
---
<img width="600" alt="image" src="/images/talks/BatchTransferCron.png">

Typicall workloads begin with flatfiles which are extracted from the source system which are either in CSV or Parquet. We will require to transfer these files into the cloud. The solution is very straight forward
* Python gcloud sdk to upload the files
* Cron job to schedule the job

However, many would say that this solution is not enterprise grade. I will use the architecture pillars and enhance the subsytems to deliver a system which is secure, reliable, performant and cost optimized while conforming to operational excelence standards.

#  1. Automation and Scheduling

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

# 2. Logging
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

![image](https://github.com/user-attachments/assets/f84321ef-7c82-4897-b4b1-2643047be4a4)



If there is a need to pesist the logs in a more robus soltuion, we could publish these messages to the cloud via pub/sub.

# 3. Pub/Sub Notification
We can publish logs to Cloud Pub/Sub and leverage cloud storage persist, logging and monitoring capabilities to build the needed reliability. Below are the typical logs which we would need to publish.
1. Job Started
![image](https://github.com/user-attachments/assets/64a8860c-1dfe-4281-8e19-e907b2e8e425)

2. Job Info Logs
![image](https://github.com/user-attachments/assets/d32f31ab-e678-4097-af6c-317a0c872f5b)

3. Job Complete
![image](https://github.com/user-attachments/assets/c02bd344-f80f-4547-963c-9f777b836721)

4. Job Error
![image](https://github.com/user-attachments/assets/b012010c-302d-4b16-987d-2789739c8284)

![image](https://github.com/user-attachments/assets/3d27dca3-0e91-47c7-8a27-f99e3cbd7bcb)

I have a [demo on how to publish notificaitons to Pub/Sub from an on-prem client](https://nuneskris.github.io/teaching/GCloudSDK-Storage-PubSub).

I have another [demo: Building Reliability in On-prem Data Upload Jobs Through Log Monitoring](https://nuneskris.github.io/teaching/GCP-Onprem-PubSub-CF-Monitoring) on how to monitor message using Cloud Function for Cloud Logging and use Cloud Monitoring for Alerting, Notification and Incident Management 

# 4. Retry Logic
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

# Run 

### Parquet Files Loaded in the Onprem extract source location.

![image](https://github.com/user-attachments/assets/b0105be3-9ba7-4cf8-8005-95da0838fe8c)
