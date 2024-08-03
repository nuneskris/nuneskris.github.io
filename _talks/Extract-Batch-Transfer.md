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

* * * * *
| | | | |
| | | | └─── Day of the week (0 - 7) (Sunday is both 0 and 7)
| | | └───── Month (1 - 12)
| | └─────── Day of the month (1 - 31)
| └───────── Hour (0 - 23)
└─────────── Minute (0 - 59)


# Run


### Parquet Files Loaded in the Onprem extract source location.

![image](https://github.com/user-attachments/assets/b0105be3-9ba7-4cf8-8005-95da0838fe8c)
