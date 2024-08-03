---
title: "Extract: Batch Transfer From Onprem To Cloud (GCP)"
collection: talks
permalink: /talks/Extract-Batch-Transfer
date: 2024-06-29
---

<img width="354" alt="image" src="/images/_talks/BatchTransferCron.png">

# Setting Up Data Transfer from On-Premises to Cloud

## Cloud Security - IAM Service Account
![image](https://github.com/user-attachments/assets/716b99cf-74de-449c-8121-a2dcdc24f455)

We woud need to create service account which will used by the client to authenticate into GCP using secret keys. This Service Account would need priviledges to create files in the destination bucket.
This key file which will include IAM Service account detailsa and security key details  will need to be download and used by the client.

<img width="187" alt="image" src="https://github.com/user-attachments/assets/ce58b726-c3d8-4f1e-b35c-315fd59e8c15">

## Create a Bucket which will be used for laoding data

I am creating a base for interest of time. I would usually have the bucket name is randomized for security reasons. Also ensure that the name right security is applied on the bucket based on the recommendation of the Raw-Layer.

![image](https://github.com/user-attachments/assets/38cbaca1-19c2-4909-a49f-ab5e593baa00)

## Schedule

Cron is a tried and tested option to schedule jobs. With proper documentation and maintenance, it can be a very reliable solution.
![image](https://github.com/user-attachments/assets/3b1349f8-1a24-49c4-9738-b3e3eccad443)

#### First we would need to find the correct Python path

    which python3

#### Edit the crontab
We would need to edit and add the schedule to the crontab
   
    crontab -e
    
#### Update the cron job entry with the correct Python path

    */5 * * * * /<path to python>/python3 /<path to code>/upload_to_gcs_scheduler.py >> /<path to log>/upload_to_gcs_scheduler.log 2>&1

#### We can verify the cron job
    crontab -l
    
<img width="1020" alt="image" src="https://github.com/user-attachments/assets/5d17d42e-67ce-414c-9eed-e5723edb8831">

# Design

##  Operational excellence
We can achieving operational excellence with a Python cron job by adopting simple but effective best practices to ensure reliability, scalability, security, and maintainability. 

### Automate Error Handling (Reliability)
Errors can happen because of network issues. There 2 points which we need to take into considertion. 
1. Transfer of Data
2. Sending Notificaiton on the the job.

#### Retry Logic
Implementing retry logic within the Python script to handle transient errors is straight forward. This can be done using libraries like tenacity for retries.

```python
@tenacity.retry(wait=tenacity.wait_fixed(60), stop=tenacity.stop_after_attempt(5))
def publish_message(project_id, topic_id, message):

@tenacity.retry(wait=tenacity.wait_fixed(5), stop=tenacity.stop_after_attempt(5))
def upload_to_gcs(local_file_path, bucket_name, destination_blob_name, json_credentials_path):
```

Error Logging: Log errors and successes for monitoring and debugging purposes. Ensure logs are stored persistently and can be accessed easily.


# Run 

### Parquet Files Loaded in the Onprem extract source location.

![image](https://github.com/user-attachments/assets/b0105be3-9ba7-4cf8-8005-95da0838fe8c)
