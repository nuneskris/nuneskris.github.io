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

### Changing the column order.
The new columns were created at the end. we can change the position both of the new columns back to the position where the original column was.

```python
# Altering the column Order
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems ALTER COLUMN PRODCATEGORYID AFTER SALESORDERITEM")
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems ALTER COLUMN PRODUCTITEMID AFTER PRODCATEGORYID")
df = spark.table("icebergmanagedplay.SalesOrderItems")
df.show()
```

![image](https://github.com/user-attachments/assets/57174bcf-fa78-4ead-a367-1f22530ee074)

# Partion in Iceberg

Partitioning in Iceberg helps to organize and optimize the storage of data by splitting it into more manageable pieces, known as partitions. This enhances query performance by limiting the amount of data scanned. Unlike traditional partitioning, Iceberg uses hidden partitioning, where partition columns do not need to be included in the schema. Instead, partitioning is defined separately, which makes schema evolution easier.

Partition evolution in Iceberg allows you to change the partitioning scheme of a table without rewriting the underlying data files. This feature is one of the key advantages of Iceberg over traditional partitioned tables, as it provides flexibility in how data is organized and queried over time.

Key Points about Partition Evolution in Iceberg
Non-intrusive: Partition evolution does not require rewriting the existing Parquet (or any other format) files.
Metadata Management: Iceberg manages partitioning at the metadata level, meaning it keeps track of which files belong to which partitions without modifying the files themselves.
Backward Compatibility: Old partitions remain readable even after partitioning changes, ensuring backward compatibility.
How Partition Evolution Works
When you change the partitioning scheme, Iceberg:

Updates the table metadata to reflect the new partitioning.
Writes new data using the new partitioning scheme.
Continues to read old data with the old partitioning scheme seamlessly.

```python
# Partition table based on "VendorID" column
logger.info("Partitioning table based on PRODCATEGORYID column...")
spark.sql("ALTER TABLE icebergmanagedplay.SalesOrderItems ADD PARTITION FIELD PRODCATEGORYID")
spark.sql("DESCRIBE TABLE EXTENDED icebergmanagedplay.SalesOrderItems").show()
```
![image](https://github.com/user-attachments/assets/c80b2fe5-14d7-4327-a26f-0913a9e24652)

![image](https://github.com/user-attachments/assets/d2d67206-8d2e-4410-85a7-f434e2a5dcd1)

```python
# Query table row count
count_df = spark.sql("SELECT COUNT(*) AS cnt FROM icebergmanagedplay.SalesOrderItems")
total_rows_count = count_df.first().cnt
logger.info(f"Total Rows: {total_rows_count}")
```
 MyIcebergSparkJob - INFO - Total Rows: 1930

```python
spark.sql("""
    INSERT INTO icebergmanagedplay.SalesOrderItems VALUES
    (900000000,10,'MB','1034',NULL,'USD',2499,2186.625,312.375,'I',4,'EA',DATE'2018-03-11'),
    (900000000,20,'CB','1161',NULL,'USD',399, 349.125,  49.875,'I',9,'EA',DATE'2018-03-11')
""")

# Query table row count
count_df = spark.sql("SELECT COUNT(*) AS cnt FROM icebergmanagedplay.SalesOrderItems")
total_rows_count = count_df.first().cnt
logger.info(f"Total Rows: {total_rows_count}")
```
 MyIcebergSparkJob - INFO - Total Rows after insert: 1932

 ![image](https://github.com/user-attachments/assets/fc3f8a56-72f6-46be-84bb-134a0ea3149b)

# Table Metadata Management
After inserting records we now have 2 snaphots

```python
# Check the snapshots available
logger.info("Checking snapshots...")
snap_df = spark.sql("SELECT * FROM icebergmanagedplay.SalesOrderItems.snapshots")
snap_df.show()
```
 ![image](https://github.com/user-attachments/assets/ee3c024c-c6a3-4664-81ae-8bee81428cf2)



