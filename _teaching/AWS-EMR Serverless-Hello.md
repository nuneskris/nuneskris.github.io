---
title: "EMR Serverless - Finally"
collection: teaching
type: "Lakehouse"
permalink: /teaching/AWS-EMR Serverless-Hello
date: 2024-06-01
venue: "EMR"
date: 2024-06-01
location: "AWS"
---
# Set up
1. IAM
1.1 Create a new role for a notebook via a custom trust policy: Role Name: EMRNotebookRole
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Principal": {
			    "Service":"elasticmapreduce.amazonaws.com"
			},
			"Action": "sts:AssumeRole"
		}
	]
}
```
### Add Permission
AmazonS3FullAccess
AmazonElasticMapReduceEditorsRole

1.2 Creat anothe role for EMR Serverless Execution
```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Principal": {
			    "Service":"emr-serverless.amazonaws.com"
			},
			"Action": "sts:AssumeRole"
		}
	]
}
```
Assign Permission: AWSGlueServiceRole
Role creared: emr-serverless-execution-play-role




# EMR Studio
Create a EMR Studio: For the Notebooks, I like to chose Interactive workloads "Create a Studio, a Workspace, and an EMR Serverless application with the necessary storage and permissions to store, organize, and run active notebooks."

Studio name: com-kfn-study-aws-emrserverless-play-studio
Created a bucket and selected it: s3://com.kfn.study.aws.emr.playemrbucket
Service Role for the notebook: Used the Role which I had created earlier: EMRNotebookRole
Created a serverless application: com-kfn-study-aws-emrserverless-play-application
Assigned the role to the application: emr-serverless-execution-play-role

# EMR Application
An application with name com-kfn-study-aws-emrserverless-play-application is available within the Studio which was launched.
The details of the application which was created are:
Type: Spark
Release version: emr-6.15.0
Architecture: x86_64
Number of Spark drivers: 1
Size of driver: 4 vCPUs, 16 GB memory
Driver disk details: 20 GB disk
Number of Spark executors: 2
Size of executor: 4 vCPUs, 16 GB memory
Executor disk details: 20 GB disk


