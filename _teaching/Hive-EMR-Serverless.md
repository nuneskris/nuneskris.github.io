---
title: "Hive on EMR Serverless"
collection: teaching
type: "DataLake"
permalink: /teaching/Hive-EMR-Serverless
date: 2024-06-01
venue: "Glue"
date: 2024-06-01
location: "AWS"
---

AWS EMR Serverless is magic to launch Hive. I remember spending a week mnay years back to get Hive working on my local machine. This is so easy on AWS, it is cheating!

# Pre-setup

## Create EMR Access Role
Create IAM role for the EMR using custom trust policies
```json
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "elasticmapreduce.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Attach Policies
* AmazonElasticMapReduceEditorsRole 
* AmazonS3FullAccess

## Create EMR Servlerless Execution Role

Create a Policy for the role: KFNStudyEMRServlerlessExecutionRolePolicies
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ReadAccessForEMRSamples",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::*.elasticmapreduce",
                "arn:aws:s3:::*.elasticmapreduce/*"
            ]
        },
        {
            "Sid": "FullAccessToOutputBucket",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::*",
                "arn:aws:s3:::*/*"
            ]
        },
        {
            "Sid": "GlueCreateAndReadDataCatalog",
            "Effect": "Allow",
            "Action": [
                "glue:GetDatabase",
                "glue:CreateDatabase",
                "glue:GetDataBases",
                "glue:CreateTable",
                "glue:GetTable",
                "glue:UpdateTable",
                "glue:DeleteTable",
                "glue:GetTables",
                "glue:GetPartition",
                "glue:GetPartitions",
                "glue:CreatePartition",
                "glue:BatchCreatePartition",
                "glue:GetUserDefinedFunctions"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Create IAM role using custom trust policie: KFNStudyEMRServlerlessExecutionRole
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "emr-serverless.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
## Create a bucket for Hive DB
Bucket Name: emr-hive-serverless-kfn-study
Folder: hive/scripts, hive/lake/employee

## EMR Servless Studio Setup
I am creating a default Hive application:
![image](https://github.com/user-attachments/assets/5531d1b2-a0dd-4f7e-b7c9-e703c72dd28f)

# Create DB and Insert Data
```sql
-- create database
CREATE DATABASE IF NOT EXISTS emrdb;

-- create table; 
CREATE EXTERNAL TABLE emrdb.employee
    (
    `id` 	INT, 	
    `name` 	STRING,	
     `salary`   STRING
    )
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://emr-hive-serverless-kfn-study/inputdata/'
TBLPROPERTIES ('skip.header.line.count'='1')
;
```

The DB Scripts are saved in a folder which needs to be defined in the job.
![image](https://github.com/user-attachments/assets/8e554f8d-21df-4a70-b4a5-392f82705175)

# Sample data 
Updaded data in the above location: s3://emr-hive-serverless-kfn-study/inputdata/

![image](https://github.com/user-attachments/assets/828b4d29-991d-4430-b6f0-43bd6c7ef63b)

# Create a job
1. Attach the execution role we had created.
2. Provide the job configuration

```json
{
    "applicationConfiguration": [
        {
            "classification": "hive-site",
            "configurations": [],
            "properties": {
                "hive.exec.scratchdir": "s3://emr-hive-serverless-kfn-study/hive/datalake/employee/scratch",
                "hive.metastore.warehouse.dir": "s3://emr-hive-serverless-kfn-study/hive/datalake/employee/data"
            }
        }
    ],
    "monitoringConfiguration": {}
}
```

![image](https://github.com/user-attachments/assets/0f071377-0481-458a-9d63-1746cfc75175)

# Validate Job

Validating the data in the S3.
![image](https://github.com/user-attachments/assets/64f8d258-a498-47ea-b74d-7a2a6ae8412c)

The Table is available in AWS Glue
![image](https://github.com/user-attachments/assets/54f74020-8bd6-4962-a436-3e9aa01e831f)

The data can additionally queried in Athena
![image](https://github.com/user-attachments/assets/b56f25dd-57b3-4b73-9ef8-6e668bb48d9d)
