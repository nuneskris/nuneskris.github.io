---
title: "DataFlow: Source Storage, Transform and Sink Biquery"
collection: teaching
type: "Data Processing"
permalink: /teaching/DataFlowStorageBigquery
date: 2024-06-01
venue: "GCP"
date: 2024-06-01
location: "GCP"
---

We will use a very simple but higly used dataflow job type I have seen. Move data from Cloud Storage to Bigquery with a few Transform Steps.

# 1. Source: Cloud Storage

![image](https://github.com/user-attachments/assets/60052dfa-06e0-4f15-85fd-d58653da3f08)

We can configure the Source Job as below.

![image](https://github.com/user-attachments/assets/054b4817-39e6-493e-b0c5-d93e6499609f)

# 2. Target: Big Query
I created a schema and table in BigQuery.

![image](https://github.com/user-attachments/assets/d8639512-d62e-40a4-8c5d-f5103eab4ad3)

The Sink for the job is accordingly configured.

![image](https://github.com/user-attachments/assets/968a85d1-29a1-4fb1-bce2-e6f5af8b1e37)


# 3. Transform
This is simple task for filtering rows.

![image](https://github.com/user-attachments/assets/504973a7-3ba5-4dd1-97fb-8cba5118d9c8)

# The Run
