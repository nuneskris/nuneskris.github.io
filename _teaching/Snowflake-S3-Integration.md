---
title: "Snowflake: AWS S3 Loading"
collection: teaching
type: "Datawarehouse"
permalink: /teaching/Snowflake-S3-Integration
venue: "S3"
date: 2024-06-01
location: "Snowflake"
---

<img width="354" alt="image" src="/images/teachings/snowflake/snowflakesetup.png"> 

I had used the Template SQL Worksheet provided by Snowflake with an end to end process to load data from Amazon S3 which I play around with. It worked like a champ. Also used the Snowflake [documentation](https://docs.snowflake.com/en/sql-reference/functions/system_validate_storage_integration).

# Setup
```sql

/*--

Helpful Snowflake Documentation:
        1. Bulk Loading from Amazon S3 - https://docs.snowflake.com/en/user-guide/data-load-s3
        2. https://docs.snowflake.com/en/sql-reference/functions/system_validate_storage_integration
-*/
-------------------------------------------------------------------------------------------
    -- Step 1: To start, let's set the Role and Warehouse context
        -- USE ROLE: https://docs.snowflake.com/en/sql-reference/sql/use-role
        -- USE WAREHOUSE: https://docs.snowflake.com/en/sql-reference/sql/use-warehouse
-------------------------------------------------------------------------------------------

--> To run a single query, place your cursor in the query editor and select the Run button (⌘-Return).
--> To run the entire worksheet, select 'Run All' from the dropdown next to the Run button (⌘-Shift-Return).

---> set Role Context
USE ROLE accountadmin;

---> set Warehouse Context
USE WAREHOUSE compute_wh;
-------------------------------------------------------------------------------------------
    -- Step 2: Create Database
        -- CREATE DATABASE: https://docs.snowflake.com/en/sql-reference/sql/create-database
-------------------------------------------------------------------------------------------
---> create the Database
CREATE  OR REPLACE  DATABASE  db_prestage;
-------------------------------------------------------------------------------------------
    -- Step 3: Create Schema
        -- CREATE SCHEMA: https://docs.snowflake.com/en/sql-reference/sql/create-schema
-------------------------------------------------------------------------------------------
---> create the Schema
CREATE SCHEMA IF NOT EXISTS db_prestage.ERP
   COMMENT = 'Loading data from S3' ;
-------------------------------------------------------------------------------------------
    -- Step 4: Create Table
        -- CREATE TABLE: https://docs.snowflake.com/en/sql-reference/sql/create-table
-------------------------------------------------------------------------------------------
---> create the Table
CREATE  TABLE  IF NOT EXISTS  db_prestage.ERP.business_partners
    (
   PARTNERID INTEGER,
    PARTNERROLE INTEGER,
    EMAILADDRESS varchar,
    PHONENUMBER INTEGER,
    FAXNUMBER varchar,
    WEBADDRESS varchar,
    ADDRESSID INTEGER,
    COMPANYNAME varchar,
    LEGALFORM varchar,
    CREATEDBY INTEGER,
    CREATEDAT INTEGER,
    CHANGEDBY INTEGER,
    CHANGEDAT INTEGER,
    CURRENCY varchar
    --> supported types: https://docs.snowflake.com/en/sql-reference/intro-summary-data-types.html
    )
    COMMENT = 'Creating a table';

---> query the empty Table
SELECT * FROM db_prestage.ERP.business_partners;
```

Follow the instructions which was strainght forward. This was the IAM JSON policy which I had created and worked for me.
```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:GetObjectVersion",
                "s3:DeleteObject",
                "s3:DeleteObjectVersion"
            ],
            "Resource": "arn:aws:s3:::com-kfn-landing-s3storage-play-erp/erp/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::com-kfn-landing-s3storage-play-erp",
            "Condition": {
                "StringLike": {
                    "s3:prefix": [
                        "erp/*"
                    ]
                }
            }
        }
    ]
}
```

# Create a Snowflake Integration
```sql
-------------------------------------------------------------------------------------------
    -- Step 5: Create Storage Integrations
        -- CREATE STORAGE INTEGRATION: https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration
-------------------------------------------------------------------------------------------

    /*--
      A Storage Integration is a Snowflake object that stores a generated identity and access management
      (IAM) entity for your external cloud storage, along with an optional set of allowed or blocked storage locations
      (Amazon S3, Google Cloud Storage, or Microsoft Azure).
    --*/

---> Create the Amazon S3 Storage Integration
    -- Configuring a Snowflake Storage Integration to Access Amazon S3: https://docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration

CREATE  OR REPLACE STORAGE INTEGRATION kfn_s3_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::573981122510:role/SnowflakeS3ReadRole'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('s3://com-kfn-landing-s3storage-play-erp/erp/' )
 ;

    /*--
      Execute the command below to retrieve the ARN and External ID for the AWS IAM user that was created automatically for your Snowflake account.
      You’ll use these values to configure permissions for Snowflake in your AWS Management Console:
          https://docs.snowflake.com/en/user-guide/data-load-s3-config-storage-integration#step-5-grant-the-iam-user-permissions-to-access-bucket-objects
    --*/

DESCRIBE INTEGRATION kfn_s3_integration;
```
<img width="848" alt="image" src="https://github.com/user-attachments/assets/e1840c98-3a4a-456e-bf7f-c922c93c51d3">

```sql
SHOW STORAGE INTEGRATIONS;
```
![image](https://github.com/user-attachments/assets/b336661d-5e8c-4c30-8b24-5d6ab53bf5e7)

# upload data into the bucket
![image](https://github.com/user-attachments/assets/4a787425-12a1-4b33-bf10-5ebbea50c1c4)

# Create a stage
```sql

-------------------------------------------------------------------------------------------
    -- Step 6: Create Stage Objects
-------------------------------------------------------------------------------------------

    /*--
      A stage specifies where data files are stored (i.e. "staged") so that the data in the files
      can be loaded into a table.
    --*/

---> Create the Amazon S3 Stage
    -- Creating an S3 Stage: https://docs.snowflake.com/en/user-guide/data-load-s3-create-stage
    
create stage kfn_s3_stage
 storage_integration = kfn_s3_integration
 url = 's3://com-kfn-landing-s3storage-play-erp/erp/'
 ;

---> View our Stages
    -- SHOW STAGES: https://docs.snowflake.com/en/sql-reference/sql/show-stages
```
![image](https://github.com/user-attachments/assets/1bfba479-9435-4cd1-b7bc-c7ba01530305)

```sql
list @kfn_s3_stage;
```
![image](https://github.com/user-attachments/assets/b0f954fd-1460-4824-b560-39f9a43fd4e4)

# Load Data
```sql
-------------------------------------------------------------------------------------------
    -- Step 7: Load Data from Stages
-------------------------------------------------------------------------------------------

---> Load data from the Amazon S3 Stage into the Table
    -- Copying Data from an S3 Stage: https://docs.snowflake.com/en/user-guide/data-load-s3-copy
    -- COPY INTO <table>: https://docs.snowflake.com/en/sql-reference/sql/copy-into-table

COPY INTO db_prestage.ERP.business_partners
  FROM @kfn_s3_stage
    FILES = ( 'BusinessPartners.csv' ) 
 file_format = (type = csv skip_header = 1 field_delimiter = ',');

-------------------------------------------------------------------------------------------
    -- Step 8: Start querying your Data!
-------------------------------------------------------------------------------------------

---> Great job! You just successfully loaded data from your cloud provider into a Snowflake table
---> through an external stage. You can now start querying or analyzing the data.

SELECT * FROM db_prestage.ERP.business_partners;
```
![image](https://github.com/user-attachments/assets/c60bdac3-28ba-4cb5-bc6f-3be2e7183751)


