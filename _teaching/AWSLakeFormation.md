---
title: "AWS Lake Formation."
collection: teaching
type: "Data Lake"
permalink: /teaching/AWSLakeFormation
venue: "LakeFormation"
location: "AWS"
date: 2025-04-16
---
AWS Lake Formation = Scaled Data Lake + Scaled Security Provisioning
<img width="454" alt="image" src="https://github.com/user-attachments/assets/27a296a3-5a12-4de0-bd5c-a47c0287bedb" />

We will provision access to the users based on the mapped tags to the database
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

## Step 1: Setup the LF datalake administrators. 

We will be using the lake admin role we have been using for this.

![image](https://github.com/user-attachments/assets/a5852779-6726-4afa-9ca7-850cdefd99d9)

## Step 2: Data Lake Location

We can use Data Location as an additional layer of securing roles having access to the data.
![image](https://github.com/user-attachments/assets/d934154f-7e2f-4266-b5a8-dbc03fcd46f3)

## Step 3: Setting up Tags
We will use tags as an additional layer of securing roles having access to the data.
![image](https://github.com/user-attachments/assets/8cf57a36-7998-43f6-85ec-30d465b131f8)

## Step 4: Setting up permission of Tags to Tables

![image](https://github.com/user-attachments/assets/b0e755ff-d94e-465e-9cba-268e527b2f0c)

![image](https://github.com/user-attachments/assets/bba31d35-4e93-4a09-aa1b-fd611fcfbb74)

## Step 5: Setting up permissions of Data Permissions of User based on Tags
Now that we have setup tables and associated the tags for access to the tables, we now would need to associate the tags to users so that users can access to the ables based on matched tags.
![image](https://github.com/user-attachments/assets/89561f3d-e931-4797-a6db-388638a09c15)

# Testing LF Analyst
Only LF Tag Data Sensitivity: sensitive need to be visible.

Only the table "balls" is available to the LF Analyst user. The match table is not available.

![image](https://github.com/user-attachments/assets/810142d6-a976-4707-ad2e-724379784d41)

# Testing LF Enginer
Providing access to tables and databased with Tag DataSensitivity: internal
![image](https://github.com/user-attachments/assets/1b99d9f2-45d5-4b17-b5b0-6d22cafbb00a)

The engineer does not have access to the balls table.
![image](https://github.com/user-attachments/assets/93bce54d-0666-4eb1-b744-652312296c83)

But the engineer does have access to the Match table
![image](https://github.com/user-attachments/assets/c42d7b79-1aa3-4655-b663-b83a9406fd58)

# Some more interesting play

The database is set with tag value: sensitive. But I did override the Match with tag value internal. That is why the value is mentioned as overridden in the match table.

![image](https://github.com/user-attachments/assets/41f6fec5-7e35-4fb0-aef4-8cd64e65dfec)

The implication for this is that, the database describe is not assessable for the engineer but the Match table is query-able. 

![image](https://github.com/user-attachments/assets/7995382d-3eee-44da-9327-cc461dee373c)




