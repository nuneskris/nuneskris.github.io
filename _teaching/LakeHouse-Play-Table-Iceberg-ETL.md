---
title: "Spark ETL on Iceberg"
collection: teaching
type: "Lakehouse"
permalink: /teaching/LakeHouse-Play-Table-Iceberg-ETL
date: 2024-07-01
venue: "Iceberg, Spark"
date: 2024-06-01
location: "Docker"
---

This is a continuation from the post which focused on [Setup, Table Management and Schema Evolution of Iceberg](https://nuneskris.github.io/teaching/LakeHouse-Play-Table-Iceberg)

The main reason for using Spark on Iceberg, is for it to provide ETL services on a table.

# Tranformation

# 1. Changing the column type. from int (yyyymmdd) to date. (Change Table Name, Drop a table
This was a bit tricky. 
```python
from pyspark.sql.functions import col, to_date
df = spark.table("icebergmanagedplay.SalesOrderItems")

# Transform the DELIVERYDATE column to date format
transformed_df = df.withColumn("DELIVERYDATE", to_date(col("DELIVERYDATE").cast(StringType()), "yyyyMMdd"))

# Create a new table with the desired schema
transformed_df.writeTo("icebergmanagedplay.SalesOrderItems_temp").create()

# Optionally, rename the new table to the original table name.
spark.sql("ALTER TABLE SalesOrderItems RENAME TO my_table_old")

spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems_temp RENAME to SalesOrderItems")

#Completely purge the table skipping trash. 
spark.sql("DROP TABLE icebergmanagedplay.my_table_old")

spark.sql("DESCRIBE TABLE EXTENDED icebergmanagedplay.SalesOrderItems").show()
```
<img width="325" alt="image" src="https://github.com/user-attachments/assets/8c780f7a-8e12-4dae-a4a4-cee756247de7">

![image](https://github.com/user-attachments/assets/7c097bac-ee8a-4cef-9daa-228ba9d4f690)


