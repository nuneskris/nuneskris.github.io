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
1.1 Create a new role for a notebook via a custom trust policy
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
Role creared: atheemr-serverless-execution-play-role
