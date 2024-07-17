---
title: "AWS Glue Spark Evolves Iceberg Schema"
collection: teaching
type: "Lakehouse"
permalink: /teaching/AWS-Glue-ETL-Iceberg-Schema-Evolution
date: 2024-06-01
venue: "Glue Spark, Iceberg"
date: 2024-06-01
location: "AWS"
---

# Objective
1. We had used Athena to create a Iceberg Table in this [page](https://nuneskris.github.io/teaching/LakeHouse-Play-Iceberg-AWS)
2. We has used Glue ETL Spark job to read data from a S3 location and load it into the Icebarg Table in this [post](https://nuneskris.github.io/teaching/AWS-Glue-Iceberg).

Now we will demonstrate a key feature of Iceberg. Schema Evolution. The employee table we had has some mimatched datatypes which we would liek to clean up.

# Data Type Changes: String (yyyyMMdd to Date)

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
 location 's3://com-kfn-lakehouse-s3storage-play-erp/warehouse/iceberg_employee/'
 tblproperties (
 'table_type' = 'ICEBERG',
 'format' = 'parquet'
 );
```

Using the date data type for VALIDITY_STARTDATE and VALIDITY_ENDDATE is generally preferred over using int. This allows you to leverage date functions in Spark SQL for comparisons, calculations, and other date-related operations. Here’s how you can modify your table to use the date data type:

## Job Script

I checked and unfortunately, Athena itself does not currently support direct schema evolution operations for Iceberg tables. However, we can achieve this through Spark-ETL.

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql.functions import col, to_date
from pyspark.sql.types import IntegerType

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)


# Read data from Iceberg table
df = spark.read \
    .format("iceberg") \
    .load("glue_catalog.com_kfn_lakehouse_iceberg_play_erp.iceberg_employee")

# Convert datestring to date type and create a temporary new column
df = df.withColumn("VALIDITY_STARTDATE_temp", to_date(col("VALIDITY_STARTDATE"), "yyyyMMdd"))

# Overwrite the original column with the new date values
df = df.drop("VALIDITY_STARTDATE").withColumnRenamed("VALIDITY_STARTDATE_temp", "VALIDITY_STARTDATE")

# Convert datestring to date type and create a temporary new column
df = df.withColumn("VALIDITY_ENDDATE_temp", to_date(col("VALIDITY_ENDDATE"), "yyyyMMdd"))

# Overwrite the original column with the new date values
df = df.drop("VALIDITY_ENDDATE").withColumnRenamed("VALIDITY_ENDDATE_temp", "VALIDITY_ENDDATE")


# Convert datestring to date type and create a temporary new column
df = df.withColumn('EMPLOYEEID_integer', df['EMPLOYEEID'].cast(IntegerType()))
# Overwrite the original column with the new date values
df = df.drop("EMPLOYEEID").withColumnRenamed("EMPLOYEEID_integer", "EMPLOYEEID")

# Convert datestring to date type and create a temporary new column
df = df.withColumn('ADDRESSID_integer', df['ADDRESSID'].cast(IntegerType()))
# Overwrite the original column with the new date values
df = df.drop("ADDRESSID").withColumnRenamed("ADDRESSID_integer", "ADDRESSID")

# Write data back to Iceberg table
df.write \
    .format("iceberg") \
    .mode("overwrite") \
    .save("glue_catalog.com_kfn_lakehouse_iceberg_play_erp.iceberg_employee")

job.commit()
```
## Validation
      Select * from iceberg_employee;
![image](https://github.com/user-attachments/assets/cc9d1efe-0da1-49cb-b409-d47a6bb775af)

> One interesting observation: The Glue schema did not automaticlly change.

![image](https://github.com/user-attachments/assets/2e8f8986-ffcd-4f4b-8fbf-c7ab64fd1743)

![image](https://github.com/user-attachments/assets/ad145380-a7b1-4ba8-bac1-7b7dadcab853)

> Addtionally I wanted to clean up the addressid and employeeid from string to int. Code is update above.

``` 
