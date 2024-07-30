---
title: "Iceberg on AWS: Part 1 - Hello World"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Iceberg-AWS
venue: "Iceberg"
location: "AWS"
date: 2024-06-01
---
<img width="354" alt="image" src="/images/teachings/iceberg/IcebergAWS.png">

You pay for what you get! Getting things started with Iceberg on AWS was a beeeze. I was actaully plesantly supprised to find out the Iceberg is a first class table format along with AWS Glue. They must be seing promise with the technology. 

# Setup
1. AWS Glue can create Iceberg Tables directily

![image](https://github.com/user-attachments/assets/c89b44a7-d8bc-4b34-b78b-d2dae3a62c6e)

2. Setup a bucket
3.  Setting up a schema
```json
[
  {
    "Name": "NAME_FIRST",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_MIDDLE",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_LAST",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "NAME_INITIALS",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "SEX",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "LANGUAGE",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "PHONENUMBER",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "EMAILADDRESS",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "LOGINNAME",
    "Type": "string",
    "Comment": ""
  },
  {
    "Name": "ADDRESSID",
    "Type": "int",
    "Comment": ""
  },
  {
    "Name": "VALIDITY_STARTDATE",
    "Type": "int",
    "Comment": ""
  },
  {
    "Name": "VALIDITY_ENDDATE",
    "Type": "int",
    "Comment": ""
  }
]
```
AWS automatically create a folder called metadata
![image](https://github.com/user-attachments/assets/98b92f80-6e8f-488b-b732-f516e06a1629)

![image](https://github.com/user-attachments/assets/b14d6fd7-7b46-4d22-a8a6-bb9b465406eb)


Use Athena Query Editor to interact with the table (Note: I adjuested some names.
```sql
DESCRIBE AWSDataCatalog.com_kfn_study_aws_iceberg_play_database_erp.employees;

INSERT INTO AWSDataCatalog.com_kfn_study_aws_iceberg_play_database_erp.employees VALUES (
    'name_first', 'name_middle', 'name_last', 'name_initials', 'sex',
    'language', 'phonenumber', 'emailaddress', 'loginname', 
    1234345, 20241010, 20241212 
);

SELECT * FROM AWSDataCatalog.com_kfn_study_aws_iceberg_play_database_erp.employees;
```
![image](https://github.com/user-attachments/assets/5b7de8a9-4490-4e94-aad7-0cde9aca20a4)

There is a new folder created in the bucket along with the metadata folder

![image](https://github.com/user-attachments/assets/7f3d0ca8-002b-4c5e-a46d-56be02cbdfbb)

![image](https://github.com/user-attachments/assets/44dff1dd-55ec-45a1-b13d-1b42b6016a8a)
