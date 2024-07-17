---
title: "Slowly Changing Dimensions - Type 2"
collection: talks
type: "Data Modeling"
permalink: /talks/Slowly-Changing-Dimensions
date: 2024-03-01
venue: "SCD2"
date: 2024-06-01
location: "AWS"
---

## General Best Practices for SCD Implementatio*
* ****Alignment with Business*** Ensure the business undertand the SCD rules for handling historical data and changes so that they understand and use dimensional data for reporting.
There was major escalation in the reports providing wrong answers. After investigating it was uncovered that there was discrepencies in how SCD2 was developed and how BI reports consumed them.
Documenting and naming the SCD related columns clearly so that folks understand how to use them goes a long way. (VALIDITY_STARTDATE, VALIDITY_ENDDATE).
* For large dimensions (I had one for product with more than 10B rows, I struggled with performance), we would need to partitions.

* For Type 2, this includes effective start and end dates and use Appropriate Data Types: (DATE or TIMESTAMP) for start and end dates. Avoid using string representations of dates for these columns. This helps in leveraging multiple functions to make like easier.
* Use bulk operations where possible to handle large volumes of data changes to efficiently handle Large Data Volumes:
* 
* For Dimensions which change very often (like the one I worked on and struggled), have a process to archive data which may not be needed.


# Demo

##set up
We will be using the DataLakehouse use case which can be followed with.

```python
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql.functions import col, to_date, current_date, lit

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# New data (e.g., from a source system or a DataFrame)
new_data = [
 (2,      'Philipp',     'T',         'Egger-Up',  'X',           'M',     'F',    '603-610-2464',    'philipp.egger@itelo.info', 'philippm',    1000000002),
 (1,      'Derrick',     'L',         'Magill-Up', 'M',           'M',     'T',     '630-374-0306',    'derrick.magill@itelo.info','derrickm',  1000000001)
]

columns= [
'EMPLOYEEID','NAME_FIRST','NAME_MIDDLE','NAME_LAST','NAME_INITIALS', 'SEX', 'LANGUAGE', 'PHONENUMBER', 'EMAILADDRESS',             'LOGINNAME', 'ADDRESSID']

new_df = spark.createDataFrame(new_data, columns)

# Read current data from Iceberg table
current_df = spark.read \
    .format("iceberg") \
    .load("glue_catalog.com_kfn_lakehouse_iceberg_play_erp.iceberg_employee") \
    .filter(col("VALIDITY_ENDDATE") == '9999-12-31')\
    .withColumnRenamed("NAME_FIRST", "CURRENT_NAME_FIRST") \
    .withColumnRenamed("NAME_MIDDLE", "CURRENT_NAME_MIDDLE") \
    .withColumnRenamed("NAME_LAST", "CURRENT_NAME_LAST") \
    .withColumnRenamed("NAME_INITIALS", "CURRENT_NAME_INITIALS") \
    .withColumnRenamed("SEX", "CURRENT_SEX") \
    .withColumnRenamed("LANGUAGE", "CURRENT_LANGUAGE") \
    .withColumnRenamed("PHONENUMBER", "CURRENT_PHONENUMBER") \
    .withColumnRenamed("EMAILADDRESS", "CURRENT_EMAILADDRESS") \
    .withColumnRenamed("LOGINNAME", "CURRENT_LOGINNAME") \
    .withColumnRenamed("ADDRESSID", "CURRENT_ADDRESSID")

# Join new data with current data to identify changes
joined_df = new_df.join(current_df, on="EMPLOYEEID", how="left") \
    .withColumn("VALIDITY_STARTDATE", current_date()) \
    .withColumn("VALIDITY_ENDDATE", lit('9999-12-31').cast("date"))

# Filter rows that have changed
changes_df = joined_df.filter(
    (new_df["NAME_FIRST"] != current_df["CURRENT_NAME_FIRST"]) |
    (new_df["NAME_MIDDLE"] != current_df["CURRENT_NAME_MIDDLE"]) |
    (new_df["NAME_LAST"] != current_df["CURRENT_NAME_LAST"]) |
    (new_df["NAME_INITIALS"] != current_df["CURRENT_NAME_INITIALS"]) |
    (new_df["SEX"] != current_df["CURRENT_SEX"]) |
    (new_df["LANGUAGE"] != current_df["CURRENT_LANGUAGE"]) |
    (new_df["PHONENUMBER"] != current_df["CURRENT_PHONENUMBER"]) |
    (new_df["EMAILADDRESS"] != current_df["CURRENT_EMAILADDRESS"]) |
    (new_df["LOGINNAME"] != current_df["CURRENT_LOGINNAME"]) |
    (new_df["ADDRESSID"] != current_df["CURRENT_ADDRESSID"])
)

# Update the current records to set validity_enddate
updated_current_df = current_df.alias("current").join(
    changes_df.select("EMPLOYEEID").alias("changes"), on="EMPLOYEEID", how="inner"
).withColumn("VALIDITY_ENDDATE", current_date())

# Union the new records with the updated current records
final_df = changes_df.select("EMPLOYEEID", "NAME_FIRST", "NAME_MIDDLE", "NAME_LAST", 'NAME_INITIALS', 'SEX', 'LANGUAGE', 'PHONENUMBER', 'EMAILADDRESS', 'LOGINNAME', 'ADDRESSID', "VALIDITY_STARTDATE", "VALIDITY_ENDDATE") \
    .union(updated_current_df.select("EMPLOYEEID", 
            col("CURRENT_NAME_FIRST").alias("NAME_FIRST"), 
            col("CURRENT_NAME_MIDDLE").alias("NAME_MIDDLE"), 
            col("CURRENT_NAME_LAST").alias("NAME_LAST"), 
            col("CURRENT_NAME_INITIALS").alias("NAME_INITIALS"), 
            col("CURRENT_SEX").alias("SEX"), 
            col("CURRENT_LANGUAGE").alias("LANGUAGE"), 
            col("CURRENT_PHONENUMBER").alias("PHONENUMBER"), 
            col("CURRENT_EMAILADDRESS").alias("EMAILADDRESS"), 
            col("CURRENT_LOGINNAME").alias("LOGINNAME"), 
            col("CURRENT_ADDRESSID").alias("ADDRESSID"), 
            "VALIDITY_STARTDATE", 
            "VALIDITY_ENDDATE"
            ))

# Write the final dataframe back to the Iceberg table
final_df.write \
    .format("iceberg") \
    .mode("append") \
    .save("glue_catalog.com_kfn_lakehouse_iceberg_play_erp.iceberg_employee")
job.commit()
```
