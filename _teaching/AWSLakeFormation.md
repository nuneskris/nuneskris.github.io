---
title: "AWS Lake Formation."
collection: teaching
type: "Data Lake"
permalink: /teaching/AWSLakeFormation
venue: "LakeFormation"
location: "AWS"
date: 2025-04-16
---

# IAM Setup

![image](https://github.com/user-attachments/assets/ff11bc15-0ff1-4498-b436-2b673fb17095)

## Data Lake Adminstrator
kfn-lf-admin
These are few of policies which we will need by the lake admin. Apart from Data Lake Admin who will configure the policies on the lake, we are going to use Athena, Redshift and Glue so that we can distunguish between the other roles which will configure.

![image](https://github.com/user-attachments/assets/1bb4eebd-f431-4806-83bf-493ef40f4cfa)

### Service Linked Role
A service-linked role is a special type of AWS IAM role that is predefined and managed by an AWS service, allowing that service to perform actions on our behalf.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": "lakeformation.amazonaws.com"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PutRolePolicy"
            ],
            "Resource": "arn:aws:iam::528454491151:role/aws-service-role/lakeformation.amazonaws.com/AWSServiceRoleForLakeFormationDataAccess"
        }
    ]
}
```

## Analyst and Engineer Roles
Analyst
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lakeformation:GetDataAccess",
                "glue:GetTable",
                "glue:GetTables",
                "glue:SearchTables",
                "glue:GetDatabase",
                "glue:GetDatabases",
                "glue:GetPartitions",
                "lakeformation:GetResourceLFTags",
                "lakeformation:ListLFTags",
                "lakeformation:GetLFTag",
                "lakeformation:SearchTablesByLFTags",
                "lakeformation:SearchDatabasesByLFTags"
            ],
            "Resource": "*"
        }
    ]
}
```

Data Engineer. May be we can down select even further
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lakeformation:GetDataAccess",
                "lakeformation:GrantPermissions",
                "lakeformation:RevokePermissions",
                "lakeformation:BatchGrantPermissions",
                "lakeformation:BatchRevokePermissions",
                "lakeformation:ListPermissions",
                "lakeformation:AddLFTagsToResource",
                "lakeformation:RemoveLFTagsFromResource",
                "lakeformation:GetResourceLFTags",
                "lakeformation:ListLFTags",
                "lakeformation:GetLFTag",
                "lakeformation:SearchTablesByLFTags",
                "lakeformation:SearchDatabasesByLFTags",
                "lakeformation:GetWorkUnits",
                "lakeformation:GetWorkUnitResults",
                "lakeformation:StartQueryPlanning",
                "lakeformation:GetQueryState",
                "lakeformation:GetQueryStatistics"
            ],
            "Resource": "*"
        }
    ]
}
```

# Data Lake Setup

Using the Data Lake Admin just to show all the tables are available for query

## S3

We are creating 2 tables.

![image](https://github.com/user-attachments/assets/9115c798-4ff3-488f-a02a-45d15437e731)

## AWS Glue  Crawler
![image](https://github.com/user-attachments/assets/da1869fe-7cae-403b-8540-e81843a35d74)

## AWS Glue Database / Tables
![image](https://github.com/user-attachments/assets/e5c180af-b787-4017-87dc-1a01b17ce31d)

## Querying the Data lake via Athena
![image](https://github.com/user-attachments/assets/fe6d2b65-1aa2-439d-af74-201512c2fb63)

## Querying the Data lake via Redshift
![image](https://github.com/user-attachments/assets/35bb9bdc-3195-4000-bcb7-a386f261f54a)

# Now Lake Formation




=======
# Data Lake Adminstrator

![image](https://github.com/user-attachments/assets/1bb4eebd-f431-4806-83bf-493ef40f4cfa)

