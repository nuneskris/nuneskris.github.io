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
Iceberg enforces schema consistency, and modifying columns directly can cause conflicts.
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

## Column Transformation: Splitting a Column

To split a column with values separated by a dash (-) into two new columns and remove the old column using PySpark, you can follow these steps:

1. Create a new table with the updated schema.
2. Insert the transformed data into the new table.
3. Optionally, rename the new table to the original table name.

```python
from pyspark.sql.functions import split, col
# Load the Iceberg table
df = spark.table("icebergmanagedplay.SalesOrderItems")

# Split the column and create new columns
split_col = split(df["PRODUCTID"], "-")
transformed_df = df.withColumn("PRODCATEGORYID", split_col.getItem(0)) \
                   .withColumn("PRODUCTITEMID", split_col.getItem(1)) \
                   .drop("PRODUCTID")

# Create a new table with the updated schema
transformed_df.writeTo("icebergmanagedplay.SalesOrderItemsmy_table_new").create()

# Optionally, rename the tables
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems RENAME TO my_table_old")
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItemsmy_table_new RENAME TO SalesOrderItems")

#Completely purge the table skipping trash. 
spark.sql("DROP TABLE icebergmanagedplay.my_table_old")
```

<img width="347" alt="image" src="https://github.com/user-attachments/assets/7d7a3de9-eb15-46af-95b9-c2e8a9f463da">

![image](https://github.com/user-attachments/assets/effa5423-0494-4480-827d-4d59c14c0070)





