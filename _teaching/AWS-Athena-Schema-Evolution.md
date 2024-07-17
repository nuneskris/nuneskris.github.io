---
title: "AWS Athena Schema Evolution"
collection: teaching
type: "Lakehouse"
permalink: /teaching/AWS-Athena-Schema-Evolution
date: 2024-06-01
venue: "Athena"
date: 2024-06-01
location: "AWS"
---

# Objective
1. We had used Athena to create a Iceberg Table in this [page](https://nuneskris.github.io/teaching/LakeHouse-Play-Iceberg-AWS)
2. We has used Glue ETL Spark job to read data from a S3 location and load it into the Icebarg Table in this [post](https://nuneskris.github.io/teaching/AWS-Glue-Iceberg).

Now we will demonstrate a key feature of Iceberg. Schema Evolution. The employee table we had has some mimatched datatypes which we would liek to clean up.

# Data Type Changes

Below is the current table
```sql
Create table iceberg_employee
(
      EMPLOYEEID string,
      NAME_FIRST string,
      NAME_MIDDLE string,
      NAME_LAST string,
      NAME_INITIALS string,
      SEX string,
      LANGUAGE string,
      PHONENUMBER string,
      EMAILADDRESS string,
      LOGINNAME string,
      ADDRESSID string,
      VALIDITY_STARTDATE string,
      VALIDITY_ENDDATE string )
 location 's3://com-kfn-lakehouse-s3storage-play-erp/warehouse/employee/'
 tblproperties (
 'table_type' = 'ICEBERG',
 'format' = 'parquet'
 );
```

Using the date data type for VALIDITY_STARTDATE and VALIDITY_ENDDATE is generally preferred over using int. This allows you to leverage date functions in Spark SQL for comparisons, calculations, and other date-related operations. Here’s how you can modify your table to use the date data type:

I checked and unfortunately, Athena itself does not currently support direct schema evolution operations for Iceberg tables. However, we can achieve this through Spark-Glue ETL.



